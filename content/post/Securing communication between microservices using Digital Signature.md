---
title: "Securing Communication Between Microservices using Digital Signature"
date: 2019-12-01T19:16:47+05:30
draft: true
tags: [
    "Microservices",
    "Security",
    "Digital Signature"
]
---
This blog explains the usage of digital signature in achiving secured communication.

## Digital signature:
Using digital signature ensures
<ul>
<li>The message is sent by a legitimate source (assuming the sender's private key not compromised)</li>
<li>The message is not tampered in transit.</li>
<li>The sender cannot deny that s/he sent the message (Non-repudiation) </li>
</ul>

Digital signature does not ensure
<ul>
<li>Message body encryption and hence the confidentiality of the content</li>
</ul>

<h3> How stuff works: </h3>
<ul>
<li>The sender has stored it's X509 private, public key pair in it's keystore</li>
<li>While sending the message, the sender encrypts the message body using the it's <b> priavte key </b> and appends the output to the message. </li>
<li>On the reciever side, by referring to a trust store the consumer identifies the associated certificate and uses it to validate the signature. 
</ul>

<h3> Example: </h3>
Let's assume 
