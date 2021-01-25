---
title: "Consumer Driven Contract Tests - Part 4"
date: 2021-01-25T16:53:02+05:30
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
In this blog series, we have covered 
<ol>
<li><a href="https://swapnilsankla.me/post/consumer-driven-contract-tests-part-1/">Introduction</a></li>
<li><a href="https://swapnilsankla.me/post/consumer-driven-contract-tests-part-2/">Pact based Contract tests for services communicating in synchronous way </a></li>
<li><a href="https://swapnilsankla.me/post/consumer-driven-contract-tests-part-3/">Pact based Contract tests for services communicating in asynchronous way </a></li>
</ol>

Let's jump to the next topic. Pact broker!

So far what we have seen is consumer publishes the contract to the local directory. Provider picks up the contract from this directory. This is the simplest way of sharing the contract. And arguably the most sensible while learning. However on the real projects it's not going to work out. 
Often there are multiple people, teams involved in the development, hence we need a centralized place to host all contracts. Pact foundation has addressed this with a <b>broker</b>. Broker is an entity where consumers can publish their contract and providers can publish the verification results.

<p align= "center">
<img src="/images/broker.png">
</p>

Setting up the broker locally is very easy. We will use docker-compose. Below is the configuration.
<p></p>
{{< gist SwapnilSankla 4db70e3e7d1c5bcf61bb82421fccef27 >}} 
<p></p>

Run docker-compose up command and open broker homepage (http://localhost:9292) 

<p align= "center">
<img src="/images/broker-ui.png">
</p>

Broker homepage displays a sample contract between Example API and Example App. It also displays details like when consumer published the contract
and when provider verified it. It also has Webhook status. We will cover this later.

Now we have broker running locally so let's use it. First we need to add support to publish the contract to the broker. Update the build.gradle by referring to the details below.
<p></p>
{{< gist SwapnilSankla 77e3cf734bad21ad540d7a3e29639c7c >}}
<p></p>
Basically we added a pact gradle plugin. And configured the pactPublish gradle task. Recall that consumer tests generate contracts. And those are saved inside <u>target/pacts</u> directory. We will now publish these contracts to the broker. Run pactPublish gradle task and refresh broker page. You will find your contracts registered on it. Congrats!

<p align= "center">
<img src="/images/broker-ui-1.png">
</p>

Let's update the provider tests. Recall how do we supply the contract information to the provider tests? Yes, using annotation as below.
<p></p>
{{< gist SwapnilSankla 4c955362b67ef69a63124fd231cbbd4e >}}

Replace pactFolder annotation with 
<p></p>
{{< gist SwapnilSankla 2a26529a42200ab0e840d60123badca3 >}}
 
Run the provider tests. Now even the verification result would start appearing on the broker. Click on the 'View pact matrix' button. It should show the verification information like below. 

<p align= "center">
<img src="/images/broker-ui-2.png">
</p>
 
That's it for today. We covered what is pact broker, how to set it up on the local machine using docker compose. We also covered publishing the contract, and the verification results to the broker. In the next blog we will cover how to enforce these contract tests using broker.