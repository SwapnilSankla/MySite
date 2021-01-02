---
title: "Consumer Driven Contract Tests - Part 1"
date: 2021-01-01T23:15:26+05:30
tags: [
    "Consumer driven contract tests",
    "CDC",
    "Pact",
    "Spring boot",
    "Kotlin"
]
---
This is a first blog of the consumer driven contract tests blog series. This blog series introduces the concept and demonstrates writing contract tests for a spring boot application.

In the world of microservices we often talk about the benefits of microservices. However there are prerequisites which need to be in place before opting this architecture pattern. Martin Fowler's <a href="https://martinfowler.com/bliki/MicroservicePrerequisites.html">blog</a> covers these prerequisites really well. One of the prerequisites he talks about is <a href="https://martinfowler.com/bliki/ContinuousDelivery.html">ContinuousDelivery</a>. Continuous Delivery is a software development discipline where you build software in such a way that the software can be released to production at any time. In the context of microservices, continuous delivery means any service can be released to production at any time without having dependency on the rest of the microservices. However what guarantees that deploying new microservice does not impact the application’s overall functionality? Testing, off course. However we don’t want to launch the entire application which may consist of hundreds of microservices just to test a tiny code change in one of the services. So what do we do?

Before solving the problem let’s take a step back. Why do we need other microservices running to test a tiny code change in one of the service? The answer is, services communicate with each other to achieve a common goal. And we need to make sure that the code change does not break the working functionality. It is actually sufficient to make sure that the code change does not impact the API of the service. If the API of the service is intact then we can safely assume that the consumer microservices of this particular service are going to work correctly. This is how we can avoid launching the entire application for testing change in a service. But who is going to verify that the API of the service is intact? The answer is Contract tests. Let’s take a simple example to understand this further.

<h1>Simple loan granting system</h1>
The example is intentionally kept simple to keep focus on contract tests. The ecosystem works as below.
<ol>
  <li>User applies for the loan by calling gateway API</li>
  <li>Gateway API fetches fraud status over Http</li>
  <li>Once fraud status is received then it emits a loan creation event</li>
  <li>Loan fulfilment service is expected to listen to the event and does further processing</li>
</ol>

<p align= "center">
<img src="/images/LoanSystem.jpg">
</p>

Loan Gateway, Fraud Service and Loan Fulfillment services are developed and deployed separately as micro services. We can assume Kafka is used for message passing. Below is the test pyramid for these services look like

<p align= "center">
<img src="/images/TestPyramid.jpg">
</p>

Looking at the test pyramid, one can understand that the Unit and Integration tests for each service are independent. However the functional suite is common and it’s because we need to make sure that the system as whole works when all microservices are stitched together.

Problem with this approach is that now one can argue that these micro services are not actually independent. This is due to the fact that for deploying any service one needs to run the functional suite which needs all other services running. 

Following is how a loan gateway CI pipeline would like.

<p align= "center">
<img src="/images/CIBefore.png">
</p>

What if we are able to remove the dependency on the other services? What if the build phase itself makes sure that the integration with other services is going to be fine? If we can achieve this then the pipeline would look like below.

<p align= "center">
<img src="/images/CIAfter.png">
</p>

The yellow boxes would go away. This is what we would get with Contract tests.

Contract test validates whether a service adheres it’s contract with other services or not. Contract is the agreement between a consumer service and the provider service. The communication medium between the services can be synchronous or asynchronous. Both mediums can be tested with contract tests.

<h2>Consumer driven contract tests</h2>

As the name suggests, consumer creates a contract describing it’s expectations from the provider service. Provider service runs the contract shared by the consumer and makes sure that the consumer expectations are met. If expectation is not met then the provider cannot roll out it’s change.

That’s it for now. In the next blog we will explore how to write consumer driven contract tests using <a href="https://docs.pact.io/">Pact</a> for synchronous communication.