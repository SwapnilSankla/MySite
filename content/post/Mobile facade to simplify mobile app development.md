---
title: "Mobile facade to simplify mobile app development"
date: 2017-10-20T21:08:05+05:30
tags: [
    "Mobile development",
    "iOS",
    "Android",
    "Mobile Facade",
    "BFF",
]
---

In this blog, I will walk you through the journey of evolution of the mobile facade layer which we wrote while developing iOS and Android mobile applications. These applications consume APIs written by external vendors. APIs are developed primarily to cater data to the web application. To handle the situation we applied <a href="https://en.wikipedia.org/wiki/Facade_pattern">Facade design pattern</a>. 

The low level design of the facade is as follows:
<p align="center">
<img src="/images/MobileFacade_LLD.png">
</p>

But you must be wondering why to write and maintain an extra layer. Right? To answer, let me walk you through the journey.

## Journey

As I have mentioned before, mobile apps were hitting the APIs which cater data to web application. API responses created for website were sent to mobile apps as-is. We started consuming these backend APIs and soon, we realised that we are fetching a significant amount of data which is absolutely not needed for mobile. Hence we decided to add a server-side thin component which will filter out the unnecessary before passing responses to mobile. This component is deployed on the same server where backend services are deployed.

<p align="center">
<img src="/images/MobileFacade_HLD.png">
</p>

This layer is owned by Mobile Team. The layer also helps in reducing the impact of the changes made in backend APIs. In future, entire backend API can be replaced by another API without impacting mobile applications (of course, if contract is adhered)

## Anti-corruption
The Backend APIs are not Http compliant. For example, API returns Http status code 200 for both success and failure. Response body contains a status flag which indicates if the API call is successful.

<p align="center">
<img src="/images/MobileFacade_ErrorResponse.png">
</p>

This makes client code slightly complicated as the success handler needs to check the status flag and then trigger the error handling routine if needed. This needed to be addressed by creating a shield around backend APIs. Hence, we started to transform API responses to make them HTTP compliant. Such damage control falls under category of Anti-corruption.

The intent of Anti-corruption is to either:
<p><i><font size="3" color="purple">
<ul> 
<li>Create adapters to make backend systems suitable for modern applications.</li>
<li>Hide the design issues of the backend APIs.</li>
</ul>
</font></i></p>

Martin fowler has written an amazing <a href="https://martinfowler.com/articles/refactoring-external-service.html">blog</a> about this.

## Aggregator
While implementing a feature, we came across a requirement where we needed to hit multiple backend APIs sequentially. The output of the first API was provided as an input to the next one. No user intervention was required between these calls. So making multiple calls from mobile, it actually made sense to aggregate these backend API calls in the facade.

<p align="center">
<img src="/images/MobileFacade_Aggregator.png">
</p>

Benefits:
<ul>
<li>Single vs multiple network calls.</li>
<li>Mobile does not need to know all backend endpoints.</li>
<li>If backend calls are independent, then those can be executed in parallel.</li>
</ul>

## Contract validation
Soon after we published our apps, many crashes were reported. Most of the crashes were due to failure of parsing API responses. Following were the issues:

<ul>
<li>Mandatory nodes missing</li>
<li>Data type mismatch</li>
</ul>

The root cause is the missing contract between Mobile and API Team. However, API Team refused to develop against client contracts. This led us to add a contract validation in the facade layer. If the backend response fails to adhere to the expected contract, a failure response is returned from facade to app.

## Mobile friendly transformations
Life was great after customising facade layer to support things listed above. The Facade was following REST practices. <a href="https://www.thoughtworks.com/insights/blog/rest-api-design-resource-modeling">REST</a> is about treating your domain model as resource. You query for a resource and you get it back. So simple, right? But to display the details of the resource, mobile needed to create ViewModel first. Now the creation of these viewModels was repeated on each platform. To avoid this duplication, we decided to return ViewModel as resource instead of the domain model from the facade.

<p><i><font size="3" color="purple">
Here, I am referring ViewModel as the object which is responsible for preparing and managing the data for view. It’s not the VM in MVVM.
</font></i></p>

<p align="center">
<img src="/images/MobileFacade_NoMVVM.png">
</p>

Mobile Apps started binding the received responses directly to the views. This was an important step in process of making apps lighter.

Following diagram depicts how it works. Assuming UI needs to display user’s full name on the screen. API is returning the firstName and lastName nodes separately. Facade can combine these two as per the UI need and then mobile application directly binds this to the UI label.

<p align="center">
<img src="/images/MobileFacade_MobileFriendlyTransformation.png">
</p>

<p><i><font size="3" color="purple">
Handling localisation can be tricky with this approach though.
</font></i></p>

## Links driving User capabilities

In the process of making apps lighter, one of the key consideration is how an availability of a particular feature can be controlled. Let’s take simple example of a mobile application which displays books in the library. On selecting a particular book; options like view preface, request book, claim book appears on the screen. However there are rules which drives which options are displayed.

<ol>
<li>View preface option is always displayed.</li>
<li>Request book option is displayed only when book is available in the library.</li>
<li>Else Claim book option is displayed.</li>
</ol>
Mobile can contain logic of determining which option to display. However simpler would be;the mobile facade tells the list of operations which are possible on the particular resource.

<p align="center">
<img src="/images/MobileFacade_DriveUserCapabilityViaFacade.png">
</p>

When user selects a book, mobile code reads the links of the book and then creates the buttons on runtime based on the links present. With this approach, mobile code refers to the href and makes the API call. No need to know these endpoints upfront! This means you get a flexibility of changing the actual endpoints like viewPreface, requestBook, claimBook without making any change in mobile.

<p><i><font size="3" color="purple">
No need to repeat the business logic of determining which features to be turned on/off on every platform.</font></i></p>

This is how you can leverage <a href="https://en.wikipedia.org/wiki/HATEOAS">HATEOAS</a> to make apps lighter.

## Sidekicks
<ul>
<li>Compression: <a href="http://www.gzip.org/">Gzip</a> for compressing API responses. </li>
<li>Monitoring: <a href="https://github.com/prometheus">Prometheus</a> for Service monitoring.</li>
<li>Circuit breakers: <a href="https://github.com/Netflix/Hystrix">Hystrix</a> for circuit breakers.</li>
</ul>

Looking at the offerings below, we can say Mobile facade is playing role of Backends for Frontends (BFF)

<ul>
<li>Filtering unnecessary data</li>
<li>Mobile friendly transformations</li>
<li>Links driving User capabilities</li>
<li>Aggregating backend API calls.</li>
</ul>

BFF has a simple intent of simplifying the client development. Sam Newman has explained this in depth in one of his <a href="https://samnewman.io/patterns/architectural/bff/">blog</a>.

I hope you enjoyed the journey of evolution of the facade. We started with a simple intent however with the evolution; facade is playing a vital role in the success of our apps.

But I faced one issue with BFF; response gets tightly coupled with the view. Hence you should consider this fact while designing.

## Problem: Tight coupling of a response with View

With BFF, mobile facade response gets tightly coupled with a particular view. A requirement may come wherein same backend data needs to be rendered in the different format.

Let’s take an example where backend API sends you the information for the students along with the courses they are appearing for. To make suitable for UI, you transform the response where response would be first grouped by Student and then by subject. However on another UI you might want to show response group by the subject first. Then response created first would not be suitable for the second UI.

<p align="center">
<img src="/images/MobileFacade_Problem.png">
</p>

Solution: Create separate BFF endpoints for the individual screens. Both endpoints hit the REST endpoint /students.

<p align="center">
<img src="/images/MobileFacade_Solution.png">
</p>

This blog is originally published <a href="https://medium.com/dev-data/mobile-facade-to-simplify-mobile-app-development-ab822a3ee577">here</a>.