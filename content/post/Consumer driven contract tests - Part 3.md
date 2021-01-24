---
title: "Consumer Driven Contract Tests - Part 3"
date: 2021-01-05T18:18:13+05:30
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
This is the third blog in the Consumer driven contract tests blog series. I introduced the concept in the <a href="https://swapnilsankla.me/post/consumer-driven-contract-tests-part-1/">first</a> blog. <a href="https://swapnilsankla.me/post/consumer-driven-contract-tests-part-2/">Second</a> blog covers writing contract testing using Pact for synchronous communication. In this blog, let's cover how to write contract test when communication medium is message based. 

In our example loan gateway emits the loan creation event. Loan fulfilment service listens to it and does further processing. Now in case of Http based communication, we have seen that Pact framework runs mock Http server. Message based communication can be established using many tools. <u>Unlike mock http, Pact does not launch any of them</u>, it just enables us to make sure the event consumer and event producers adhere to the same schema. Ultimately that's what we want! 

<h2>Consumer test</h2>

Let’s start with the consumer test. In our example the listener in the loan fulfilment service is the consumer of the event emitted by loan gateway. Below are the steps to generate consumer tests and the contract.

<ol>
<li>
As usual, let's start with a spring boot test. As the LoanFulfilmentConsumer class is the consumer in this case, we will write a test for it.
</li>
<li>
Let’s define the pact method first this time. We need to build MessagePact instead of RequestResponsePact. Below is the method.
<p></p>
{{< gist SwapnilSankla dcf2e7ee019ef9c4889c4cafc506346a >}}

The method is self-explanatory. We describe basically what message contains. Let’s break it down.

<p></p>
<table style="width:100%;table-layout: fixed" border="1px">
  <tr>
    <td style="padding: 2px;">@Pact(consumer ="loan_fulfilment_service", provider = "loan_gateway")</td>
    <td style="padding: 2px;">Provider and consumer names, these will be published along with the contract</td>
  </tr>
  <tr>
    <td style="padding: 2px;">expectsToReceive("Loan creation event")</td>
    <td style="padding: 2px;">This is the description of the interaction, the provider needs to provide a sample event from a method annotated with this description</td>
  </tr>
  <tr>
    <td style="padding: 2px;">withMetadata(mapOf("traceId" to "1"))</td>
    <td style="padding: 2px;">This is optional and to denote whether the message is going to contain a metadata or not</td>
  </tr>
  <tr>
    <td style="padding: 2px;">withContent(...)</td>
    <td style="padding: 2px;">Message body</td>
  </tr>   
</table>
<p></p>
</li>
<li>
The test method is supposed to receive messages. And we need to make sure that the message adheres to the message object. 
<p></p>
{{< gist SwapnilSankla efc87a19db0f381dd2ee98d78fbae915 >}}
</li>
<li>
We need to explicitly specify that the provider is going to be asynchronous. We can specify that using pactTestFor annotation. Below is the entire test.
<p></p>
{{< gist SwapnilSankla c6e37cc3ec470d8d4d7c2ca59f57113c >}}
</li>
<li>
Running this test generates the contract in the target/pacts folder.
<p></p>
{{< gist SwapnilSankla 7a6d78c07201c6a5d96f1c9f3eeeb8fb >}}
</li>
</ol>

<h2>Provider test</h2>
<ol>
<li>On the provider side we need to specify sample event which is adheres to the schema provided by the consumer.</li>
<li>Let’s start with the spring boot test and add the pact test as below. We set up the context with target as AmpqTestTarget.</li>
<p></p>
{{< gist SwapnilSankla 99a95ce4008f01620d360063472e7eed >}}
<li>Pact framework does not know from where to pick up the pact and which interactions to test from those pact files. Let’s supply that information via annotations.
<p></p>
{{< gist SwapnilSankla 13dbce8583cb68878a638f85299f5685 >}}
</li>
<li>Run the test. Test fails with an exception <i>No annotated methods were found for interaction 'Loan creation event'. You need to provide a method annotated with @PactVerifyProvider("Loan creation event") on the classpath that returns the message contents.</i></li>
<li>Looking at the string ‘Loan creation event’ makes us realise that we have mentioned this string in the expectsToReceive clause. Let’s recall the purpose. The provider needs to provide a sample event from a method annotated with this description. Let’s add a method to provide the sample event.
<p></p>
{{< gist SwapnilSankla 87955bfbee7e5ab441a06ab0d6a6321e >}}
</li>
<li>Below is the entire test.
<p></p>
{{< gist SwapnilSankla be172ecdd73cd3d5d043fd210c645fec >}}
</li>
<li>
Run the test. Now it will pass and will emit the output as below.
<p></p>
{{<highlight "linenos=table">}}
Verifying a pact between loan_fulfilment_service and loan_gateway
  [Using Directory target/pacts]
  Loan creation event
    generates a message which
      has a matching body (OK)
      has matching metadata (OK)
{{</highlight >}}
</li>
</ol>

Congratulations! Now you know how to write contract test for services communicating asynchronous. You can find entire code on <a href="https://github.com/SwapnilSankla/cdc">github</a>. 