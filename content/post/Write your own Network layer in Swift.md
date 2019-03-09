---
title: "Write your own Network layer in Swift"
date: 2018-07-10T21:08:05+05:30
tags: [
    "Swift",
    "iOS",
    "Networking",
    "RxSwift",
]
---
Let us try to build our own Network layer in pure Swift. It is super simple to build with Swift’s strong support for generics, type inference

So let’s start with basics. This network layer is based on <a href="https://developer.apple.com/documentation/foundation/urlsession">URLSession</a> object. By definition this is an object that coordinates a group of related network data transfer tasks. You don’t even need to create a custom object of URLSession in most of the cases. We will also keep it simple and use the default shared instance.

## Get Requests

To make a GET request let’s use a dataTask method on URLSession. The method looks like below.

{{< highlight swift "linenos=table">}}
open func dataTask(with request: URLRequest, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Swift.Void) -> URLSessionDataTask
{{< / highlight >}}

We need to pass following parameters:
<ol>
    <li>request: GET requests are pretty simple. We just need endpoint and header details. We can create the URLRequest with these.</li>
    <li>completionHandler: It is called as a result of request execution. It contains following fields
        <ul>
            <li>data: Data? : Contains the response in Data form. As the signature suggests, it is optional. When request succeeds you get data else it will be nil.</li>
            <li>urlResponse: URLResponse? : Contains auxiliary details like
                <ol>
                    <li>mimeType</li>
                    <li>encoding</li>
                    <li>Headers</li>
                    <li>Status code etc.</li>
                </ol>
            </li>
            <li>error: Error? : Contains error information. On success you receive it as nil.</li>
        </ul>
    </li>
</ol>
To determine whether the request has actually succeeded or not, we rely on each of these.

With above details I wrote my first network layer code.

### First attempt

{{< highlight swift "linenos=table">}}
import Foundation

class NetworkLayer {
    func get(urlRequest: URLRequest,
             completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) {
        URLSession.shared.dataTask(with: urlRequest, 
                                   completionHandler: completionHandler)
        .resume()
    }
}
{{< / highlight >}}

Bingo! It’s super simple. Make sure you call resume() method so that the suspended task gets picked up for execution.

Though the above code is simple however the caller needs to do a lot of things. It needs to take care of building and processing urlRequest and completionHandler. Most of the code which caller writes is actually common. So let’s try to simplify that.

### Second attempt
{{< highlight swift "linenos=table">}}
import Foundation

typealias NetworkCompletionHandler = (Data?, URLResponse?, Error?) -> Void
typealias ErrorHandler = (String) -> Void

class NetworkLayer {
    static let genericError = "Something went wrong. Please try again later"
    func get(urlRequest: URLRequest,
             successHandler: @escaping (Data) -> Void,
             errorHandler: @escaping ErrorHandler) {

        let completionHandler: NetworkCompletionHandler = { (data, urlResponse, error) in
            if let error = error {
                print(error.localizedDescription)
                errorHandler(NetworkLayer.genericError)
                return
            }

            if self.isSuccessCode(urlResponse) {
                guard let data = data else {
                    print("Unable to parse the response")
                    return errorHandler(NetworkLayer.genericError)
                }
                successHandler(data)
            }
            errorHandler(NetworkLayer.genericError)
        }

        URLSession.shared.dataTask(with: urlRequest,
                                   completionHandler: completionHandler)
            .resume()
    }

    private func isSuccessCode(_ statusCode: Int) -> Bool {
        return statusCode >= 200 && statusCode < 300
    }

    private func isSuccessCode(_ response: URLResponse?) -> Bool {
        guard let urlResponse = response as? HTTPURLResponse else {
            return false
        }
        return isSuccessCode(urlResponse.statusCode)
    }
}
{{< / highlight >}}

What did I change?

<ol>
    <li>Instead of completionHandler now the get method accepts success and error handlers.</li>
    <li>Success handler is called when the response contains success status code and response is convertible into Data. This makes caller code very easy. Either success or error handler gets called. Caller just need to do the processing accordingly. </li>
</ol>

Taking a closer look, I felt converting Data to appropriate response object task is also common among all consumers. So why should everyone repeat the logic? That brought me to the third attempt.

### Third attempt
{{< highlight swift "linenos=table">}}
import Foundation

class NetworkLayer {
    static let genericError = "Something went wrong. Please try again later"

    func get<T: Decodable>(urlRequest: URLRequest,
                           successHandler: @escaping (T) -> Void,
                           errorHandler: @escaping ErrorHandler) {

        let completionHandler: NetworkCompletionHandler = { (data, urlResponse, error) in
            if let error = error {
                print(error.localizedDescription)
                errorHandler(NetworkLayer.genericError)
                return
            }

            if self.isSuccessCode(urlResponse) {
                guard let data = data else {
                    print("Unable to parse the response in given type \(T.self)")
                    return errorHandler(NetworkLayer.genericError)
                }
                if let responseObject = try? JSONDecoder().decode(T.self, from: data) {
                    successHandler(responseObject)
                    return
                }
            }
            errorHandler(NetworkLayer.genericError)
        }

        URLSession.shared.dataTask(with: urlRequest, 
                                   completionHandler: completionHandler)
                  .resume()
    }
}
{{< / highlight >}}

What did I change?

<ol>
    <li>SuccessHandler signature is changed to (T) -> Void. Instead of sending Data let’s send the concrete object itself. </li>
    <li>T is restricted to be Decodable. This is the secret recipe. With Codables, we don’t need to write our own parsers to convert data to the appropriate type. </li>
</ol>

So now how does client code looks like? It’s as below.

{{< highlight swift "linenos=table">}}
import Foundation

class WorldCupMatchesPresenter {
    func get() {
        NetworkLayer().get(URLRequest(URL(string: "http://worldcup.sfg.io/matches")!),
                           successHandler: dataReceived,
                           errorHandler: dataFailedToReceive)
    }
    func success(matches: [Match]) {
        print(matches)
    }

    func error(error: String) {
        print(error)
    }
}
{{< / highlight >}}

Looking closely at caller’s code one can understand the power of <i>generics</i> and <i>type inference</i>. I don’t need to specify what network layer should return on success or on failure.

Still the creation of URLRequest can be extracted out. Let’s do that too!

### Fourth attempt

{{< highlight swift "linenos=table">}}
class NetworkLayer {
    static let genericError = "Something went wrong. Please try again later"
    func get<T: Decodable>(urlString: String,
                           headers: [String: String] = [:],
                           successHandler: @escaping (T) -> Void,
                           errorHandler: @escaping ErrorHandler) {

        let completionHandler: NetworkCompletionHandler = { (data, urlResponse, error) in
            if let error = error {
                print(error.localizedDescription)
                errorHandler(NetworkLayer.genericError)
                return
            }

            if self.isSuccessCode(urlResponse) {
                guard let data = data else {
                    print("Unable to parse the response in given type \(T.self)")
                    return errorHandler(NetworkLayer.genericError)
                }
                if let responseObject = try? JSONDecoder().decode(T.self, from: data) {
                    successHandler(responseObject)
                    return
                }
            }
            errorHandler(NetworkLayer.genericError)
        }

        guard let url = URL(string: urlString) else {
            return errorHandler("Unable to create URL from given string")
        }
        var request = URLRequest(url: url)
        request.allHTTPHeaderFields = headers
        URLSession.shared.dataTask(with: request, 
                                   completionHandler: completionHandler)
                  .resume()
    }
}
{{< / highlight >}}

What did I change?

<ol>
    <li>Consumer just needs to send the endpoint and optionally the headers. Network layer takes care of building URLRequest and handling URL creation failure.</li>
</ol>

### Error Handling

Get method calls errorHandler in one of the following cases.

<ol>
    <li>URL creation failure</li>
    <li>Response parsing failure</li>
    <li>Status code based error handling</li>
    <li>Network layer errors e.g. request timeout etc.</li>
</ol>

## POST Requests

POST requests are equally simple. To make a POST request let’s use a uploadTask method on URLSession. The method looks like below.

{{< highlight swift "linenos=table">}}
func uploadTask(with request: URLRequest, from bodyData: Data?, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionUploadTask
{{< / highlight >}}

We need to pass following parameters:

<ol>
    <li>request: URL request object.</li>
    <li>body: Data which needs to be uploaded</li>
    <li>completionHandler: We have already used this one in GET request.</li>
</ol>

As we already know the power of generics, type inference. So based on that the post method looks like below.

{{< highlight swift "linenos=table">}}
func post<T: Encodable>(urlString: String,
                        body: T,
                        headers: [String: String] = [:],
                        errorHandler: @escaping ErrorHandler) {

        let completionHandler: NetworkCompletionHandler = { (data, urlResponse, error) in
            if let error = error {
                print(error.localizedDescription)
                errorHandler(NetworkLayer.genericError)
                return
            }

            if !self.isSuccessCode(urlResponse) {
                errorHandler(NetworkLayer.genericError)
                return
            }
        }

        guard let url = URL(string: urlString) else {
            return errorHandler("Unable to create URL from given string")
        }
        var request = URLRequest(url: url)
        request.timeoutInterval = 90
        request.httpMethod = "POST"
        request.allHTTPHeaderFields = headers
        request.allHTTPHeaderFields?["Content-Type"] = "application/json"
        guard let data = try? JSONEncoder().encode(body) else {
            return errorHandler("Cannot encode given object into Data")
        }
        request.httpBody = data
        URLSession.shared
            .uploadTask(with: request,
                        from: data,
                        completionHandler: completionHandler)
            .resume()
    }
{{< / highlight >}}

Things to consider

<ol>
    <li>T is restricted to encodable and without knowing the real type we can create Data object from it. </li>
    <li>Caller does not need to take care of setting up common POST request params. Following are set inside post method.
        <ul>
            <li>headers</li>
            <li>timeout</li>
            <li>httpMethod</li>
            <li>httpBody</li>
        </ul>
    </li>
</ol>

That’s it! The Network layer now supports GET and POST requests. Although it is not very powerful however it will suffice the need.

## Unit testing
I have always seen problems while testing the code which calls network layer. So let’s check whether the caller code is testable or not.

We all know that Swift recommends to not to use mocks and rather rely on the other types of test doubles. We will use Dummy object to test caller method.

Let’s assume you want to test getExample() method from the code below.

{{< highlight swift "linenos=table">}}
class Presenter {
    let networkLayer: NetworkLayer
    var view: ViewProtocol?

    init(view: ViewProtocol,
         networkLayer: NetworkLayer = NetworkLayer()) {
        self.view = view
        self.networkLayer = networkLayer
    }

    func getExample() {
        let successHandler: ([Employee]) -> Void = { (employees) in
            print(employees)
            self.view?.displayEmployees(employees: employees)
        }
        let errorHandler: (String) -> Void = { (error) in
            print(error)
            self.view?.displayError(error: error)
        }

        networkLayer.get(urlString: "http://dummy.restapiexample.com/api/v1/employees",
                         successHandler: successHandler,
                         errorHandler: errorHandler)
    }
{{< / highlight >}}

Things to consider:
<ol>
    <li>networkLayer is injected into Presenter. This makes it easy for us to inject the dummy object.</li>
    <li>The default value of networkLayer is the real NetworkLayer. Hence the caller does not need to create and pass it.</li>
</ol>

Let’s try to write a test for getExample() method. The focus is to verify whether displayEmployees() is being called or not. We don’t want our test to make real network calls right? So time for dummy objects!

Things to consider:

<ol>
    <li>Inject the dummy network layer while creating presenter.</li>
    <li>Inject the dummy view so that we can easily test the view code and assert upon.</li>
    <li>If we want to test the success case then set networkLayer.successResponse. Otherwise set networkLayer.errorResponse.</li>
</ol>    

{{< highlight swift "linenos=table">}}
class PresenterTests: XCTestCase {
    let view = DummyViewController()
    let networkLayer = DummyNetworkLayer()

    func test_getExample_callsDisplayEmployees_onSuccess() {
        networkLayer.successResponse = [Employee(name: "Swapnil",
                                                 salary: "123456",
                                                 age: "30")]
        Presenter(view: view, networkLayer: networkLayer).getExample()
        XCTAssertTrue(view.displayEmployeesCalled)
    }

class DummyViewController: ViewProtocol {
    var displayEmployeesCalled = false

    func displayEmployees(employees: [Employee]) {
        displayEmployeesCalled = true
    }
}

public class DummyNetworkLayer: NetworkLayer {
    public var successResponse: Decodable?
    public var errorResponse: String?

    open override func get<T>(urlString: String,
                              headers: [String : String],
                              successHandler: @escaping (T) -> Void,
                              errorHandler: @escaping ErrorHandler) where T : Decodable {
        switch successResponse {
        case .some(let response):
            if let correctResponse = response as? T {
                successHandler(correctResponse)
            } else {
                errorHandler(errorResponse ?? "")
            }
        default:
            errorHandler(errorResponse ?? "")
        }
    }
}
{{< / highlight >}}

Cool! The code is easily testable, isn’t it? Similarly we can write test method for failure cases and also for the POST request.

You can find the complete code on <a href="https://github.com/SwapnilSankla/Swift_SimpleNetworkLibrary">github</a>

I have published <a href="https://cocoapods.org/pods/Swift_SimpleNetworkLibrary">Swift_SimpleNetworkLibrary</a> on cocoapods. The framework also contains the DummyNetworkLayer. Hence the unit testing becomes very easy.

To install Swift_SimpleNetworkLibrary add following into your Podfile.

```swift
pod 'Swift_SimpleNetworkLibrary'
```

## Can Rx further simplify our network layer?

Let’s try to use RxSwift to build our network layer. I am not going into the details of the Rx concepts. Let’s check the usage directly. The network layer code looks something like this.

{{< highlight swift "linenos=table">}}
func get(url: URL) -> Observable<T> {
    return Observable.create({ (observer) -> Disposable in
        let completionHandler: NetworkCompletionHandler = { (data, urlResponse, error) in
            if let error = error {
                print(error.localizedDescription)
                observer.onError(error)
                return
            }

            if let unwrappedUrlResponse = urlResponse as? HTTPURLResponse {
                if self.isSuccessCode(unwrappedUrlResponse.statusCode) {
                    if let data = data {
                        if let responseObject = try? JSONDecoder().decode(T.self, from: data) {
                            observer.onNext(responseObject)
                            return
                        } else {
                            print("Unable to parse the response in given type \(T.self)")
                        }
                    }
                }
            }
            observer.onError(NetworkError.error)
        }
        let task = URLSession.shared.dataTask(with: url,
                                              completionHandler: completionHandler)
        task.resume()
        return Disposables.create()
    })
}
{{< / highlight >}}

Things to consider:
<ol>
    <li>Returning an observable on which the caller will have to observe upon.</li>
    <li>Success and failure handlers are replaced with observer.onNext and observer.onError respectively.</li>
    <li>Complete code goes inside a closure which is passed to Observables.create</li>
</ol>

Now let’s look at the caller code.

{{< highlight swift "linenos=table">}}
func viewDidLoad() {
   disposable = networkLayer
            .get(url: URL(string: "http://worldcup.sfg.io/matches")!)
            .subscribe(onNext: dataReceived,
                       onError: dataFailedToReceive)
    }
{{< / highlight >}}

We assign onNext and onError just like we set successHandler and errorHandler earlier. Here we get a disposable object from the network layer. It needs to be disposed appropriately.

Caller code remains mostly same in both the approaches. However the get method gets complicated with Rx. Hence after trying both approaches, at least I am convinced that callbacks are simpler than Rx in current use case. Rx are more powerful however here I don’t feel the need right now.

Reference: https://swapnilsankla.github.io/Swift_SimpleNetworkLibrary/

This blog is originally published <a href="https://medium.com/dev-data/lexical-scoping-in-swift-ba95ab949b0c">here</a>.
