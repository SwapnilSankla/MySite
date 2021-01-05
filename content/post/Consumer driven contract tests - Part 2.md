---
title: "Consumer Driven Contract Tests - Part 2"
date: 2021-01-02T10:33:41+05:30
tags: [
    "Consumer driven contract tests",
    "CDC",
    "Pact",
    "Spring boot",
    "Kotlin",
    "gradle",
    "junit5"
]
---
I introduced the contract test concept in the <a href="https://swapnilsankla.me/post/consumer-driven-contract-tests-part-1">previous</a> blog. In this blog we will explore writing contract tests for services which communicate over Http. Tests are written using:
<ol>
  <li>Pact-jvm</li>
  <li>Junit5</li>
  <li>Gradle</li>
</ol>

The code is available on <a href="https://github.com/SwapnilSankla/cdc.git">github</a>.

Consumers drive contracts. They create and pass it to provider. Provider is supposed to make sure the consumer expectations are met. Provider cannot roll out it’s change if consumer's expectation are not met. Pact will help us test it.

<h2>Consumer test</h2>
Consumer tests are generally written for a class which are responsible for communicating with other service. Let’s start with the first test. Intent of this test is to cover the interaction between loan gateway and fraud service. Loan gateway and fraud service are the components which I introduced in the <a href="https://swapnilsankla.me/post/consumer-driven-contract-tests-part-1">previous</a> blog. Loan gateway passes customer id to fraud service and expects a response containing whether the given customer has fraudulent status or not. Following is a code snippet of FraudCheckService in the loan gateway service.

{{< gist SwapnilSankla 910e2dfe53a8965084f3b0703edc75b1 >}}

Let’s take a step by step approach to build the contract tests. 

<ol>
<li>Write a spring boot test which asserts that for customer id “1” the fraud status is false.
<p></p>
{{< gist SwapnilSankla c23e30f76169f6048f3d7b478558f025 >}}
</li>
<li>This test is going to fail as there is no fraud service available which is going to respond. Test passes after setting up a mock fraud service.
<p></p>
{{< gist SwapnilSankla e59589d07d06454903fb012ab5dc59ce >}}
</li>
<li>Let’s define a method which defines the contract.
<p></p>
{{< gist SwapnilSankla 6a1585a2a3b29555bdc02c2a14ae24ec >}}
This method is self-explanatory. We basically describe what the request and response are. Let’s break it down.
<p></p>
<table border="1px">
  <tr>
    <td style="padding: 2px;">@Pact(provider = "fraud_service", consumer = "loan_gateway")</td>
    <td style="padding: 2px;">Provider and consumer names, these will be published along with the contract</td>
  </tr>
  <tr>
    <td style="padding: 2px;">given("Customer with id 1 is setup to return false fraudulent status")</td>
    <td style="padding: 2px;">This is optional, it enables the provider to do test setup. We will cover this more in provider test</td>
  </tr>
  <tr>
    <td style="padding: 2px;">uponReceiving("When fraud status is requested for nonFraudulent customer")</td>
    <td style="padding: 2px;">This is the interaction description, basically it covers the scenario.</td>
  </tr>
  <tr>
    <td style="padding: 2px;">method("GET")</td>
    <td style="padding: 2px;">Http method</td>
  </tr>
  <tr>
    <td style="padding: 2px;">path("/fraud/1")</td>
    <td style="padding: 2px;">Uri</td>
  </tr>
  <tr>
    <td style="padding: 2px;">status(200)</td>
    <td style="padding: 2px;">Response status code</td>
  </tr>
  <tr>
    <td style="padding: 2px;">body(JSONObject("""{ "status": false }"""))</td>
    <td style="padding: 2px;">Response body</td>
  </tr>      
</table>
<p></p>
</li>
<li>Annotate test method with PactTestFor annotation. By this a test method connects to the pact method.</li> 
<li>Annotate test class with PactTestFor annotation. Extend the test with PactConsumerTestExt class.</li>
<li>Run the test, it will fail with <i>java.net.BindException: Address already in use </i> exception. This is because Pact itself starts the mock server. Let’s remove the mock server. Test looks like below.</li>
<p></p>
{{< gist SwapnilSankla 23a3a211c992f778675eef95be659f2b >}}
<li>Run the test. It passes now. As a nice side effect, it generates the contract at /target/pacts folder.
<p></p>
{{< gist SwapnilSankla 31cb3c15f72ac5bfe30d624a2e99534e >}}
</li>
<li>This contract is shared with provider and provider makes sure that it adheres to the consumer expectation. For easy sharing, pact provides a broker which stores the published contracts. We will cover broker topic separately.</li>
</ol>

<h2>Provider test</h2>
Provider tests are written for the controller layer. Rest of the service components are stubbed/mocked. This is because the interaction between controller and other components is already tested with the integration tests. 

Let’s start with the provider test.

<ol>
<li>Write a spring boot test for the provider service controller. In our case it is FraudServiceController. Let’s skip the other components.
<p></p>
{{< gist SwapnilSankla f4bcf61beac371a0731fd775e4758455 >}}
</li>
<li>Let’s convert the spring boot test into a provider test. We set up the context with host and port. And verify the interactions.
<p></p>
{{< gist SwapnilSankla 6468b6a3e4c887f8d2c4bcbbf54612e8 >}}
</li>
<li>
Right now pact does not know about from where to pick up the pact and which interactions to test from those pact files. Let’s supply that information via annotations.
<p></p>
{{< gist SwapnilSankla 4c955362b67ef69a63124fd231cbbd4e >}}
</li>
<li>Run the test. Test fails with an exception <i>au.com.dius.pact.provider.junit.MissingStateChangeMethod: Did not find a test class method annotated with @State("Customer with id 1 is setup to return false fraudulent status")</i></li>
<li>Looking at the string inside state annotation makes us realise that we have mentioned this string in contract under given clause. Let’s recall the purpose. It’s optional, however it enables the provider to do test setup. This information is captured under providerStates node in the contract json.</li>
<li>Let’s add a setup method, ideally this is not required as the setup is already taken care of by the mock. However as it is present in the contract it is now mandatory to define it. Consumers can skip it however I recommend to keep it as consumers may not know what data provider needs to setup. For now let’s add an empty method as below. 
<p></p>
{{< gist SwapnilSankla f54c1009ff3d835798dc59966883221f >}}
</li>
<li>The entire test looks like below.
<p></p>
{{< gist SwapnilSankla 4f534d82b62aa66c27c6395f5dc55159 >}}
</li>
<li>Run the test. Now it will pass and will emit the output as below. Last line in the output suggests that the verification results are not published even though the test is passed.
{{<highlight "linenos=table">}}

Verifying a pact between loan_gateway and fraud_service
  [Using Directory target/pacts]
  Given Customer with id 1 is setup to return false fraudulent status
  When fraud status is requested for nonFraudulent customer
  ...
    returns a response which
      has status code 200 (OK)
      has a matching body (OK)
... Skipping publishing of verification results as it has been disabled (pact.verifier.publishResults is not 'true')
{{</highlight >}}
</li>
</ol>

<h3>Publish verification results</h3>
<ol>
<li>Pact looks for an environment variable <i>pact.verifier.publishResults</i> and if it is set to true then the results are published.</li>
<li>One can set this property in the setup method of the provider tests. However this needs to be repeated for each provider test class.</li>
<li>Alternatively we can achieve this by implementing a <i>BeforeAllCallback</i> hook where we would get a handle to set this property.
<p></p>
{{< gist SwapnilSankla cabb0f8d82aaf75b07b13d89ed6eba67 >}}
</li>
<li>Extend the provider test with PactVerificationResultExtension as below
<p></p>
{{< gist SwapnilSankla b0dc962efd0e473d9d3632979305e989 >}}
</li>
<li>That’s it, now the verification results would be published.</li>
<li>Problem with this approach is, we hard coded the provider version. Now let’s take one more step and use project version specified in the build.gradle.</li>
</ol>
<h3>Refer application version from build.gradle</h3>
<ol>
<li>
Let’s use Gradle’s <i>ProcessResources</i> task. It enables us to access gradle properties in the application properties file. Add below code to build.gradle.
<p></p>
{{< gist SwapnilSankla c4747f65bc1eb5cb62bfad45b4f289cb >}}
</li>
<li>
Access the project version in application.yaml file as below.
<p></p>
{{< gist SwapnilSankla 1bde5c75e89ca08e0561c37b739d6175 >}}
</li>
<li>
Access app.version in the PactVerificationResultExtension file and set it to the pact.provider.version
<p></p>
{{< gist SwapnilSankla 710f1c72e59dd31d68dab09a0c323702 >}}
</li>
</ol>

That's it for now. In the next blog we will explore writing contract tests between services which communicates asynchronously.


