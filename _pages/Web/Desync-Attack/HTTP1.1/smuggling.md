---
title: "HTTP request smuggling"
tags:
    - HTTP Desync
    - Request Smuggling
    - HTTP/1.1
date: "2020-12-29"
thumbnail: "/assets/img/thumbnail/smuggling.gif"
---

# About the blog
---
* Author: [Nachiket Rathod](https://www.linkedin.com/in/nachiketrathod/)
* Genre: `Web Application`
* Objective: Understanding and analyzing HTTP request smuggling (HTTP Desync Attacks) and its critical security implications.

<h3 id="tldr"> 
     <strong>TL;DR: </strong>
</h3>

In this blog we will discuss the `nitty-gritty` of the **HTTP request smuggling/HTTP Desync Attacks**.
This vulnerabilities are often `critical` in nature, allowing an attacker to **bypass security controls**, gain **unauthorized access to sensitive data**, and directly compromise other application users, and the page discusses below **segments** of this vulnerability.

## Synopsis
- [Core concepts](https://nachiketrathod.com/HTTP-Desync)
- [Methodology](https://nachiketrathod.com//HTTP-Desync)
- [Detecting-desync](https://nachiketrathod.com//HTTP-Desync)
- [Confirming-desync](https://nachiketrathod.com/HTTP-Desync)
- [Explore](https://nachiketrathod.com/HTTP-Desync)

# 1. Core concepts 👻
<hr>

>**"Smashing into the Cell Next Door"**
>**"Hiding Wookiees in HTTP"**

### **What is HTTP Request Smuggling?**
<mark>It's a technique for interfering with the way of website process the sequences of HTTP requests that are received from one or more users.</mark>


 ```Text
  I divided the concept understanding into four parts.
 ```

 ```text
 - Front-end
 - Back-end
 - Content-Length
 - Transfer-Encoding
```

+ Now let's understand this vulnerability in depth. 
+ so first will see that how the `Front-end` and `Back-end` works.

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/1.gif" alt="picture" style="border:1px solid white"/>
  <figcaption style="color: red;">Fig 1. End users, Front-end and Back-end.</figcaption>
</figure><br>


1.) As an **End-users** what we can directly see?
The answer is **Front-end** part Right!, and obviously we **can't see** the **Back-end** part or it's processes.
2.) How the modern days website communicates to each other?
Well they **communicate** to each other via **chain of web-servers** speaking HTTP over stream based `transport layer` protocols like **TCP or TLS**.
3.) These streams(**TLS/TCP**) are `heavily reused` and follows the **HTTP 1.1 keepalive** protocol.


**~ `Question` - How this protocols works?**

 + As **TCP/TLS** are heavily reused that means every requests are going to placed **back to back** on this TCP/TLS-streams, and every server parse `HTTP-Headers` to identify that where each request `starts and stops.`
 + So from all over the world request are coming and passing through this tiny tunnel of **TLS/TCP** streams and passing to the **Back-end** and then **split up into individual** requests. 

 **~ `Question` - What could possibly go wrong here?**
 + <mark>what if an attacker sends an ambiguous request which is deliberately crafted and so that front-end and back-end disagree about how long this messages is.</mark><br>
**Now let's understand above line via following example,**

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/2.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 2. Attacker, Malicious prefix </figcaption>
</figure>

`Example:` <br>
 + If you see the above `Fig 2.` than **Blue block** is the orignal request and the **Orange Block** is the malicious prefix.
 + <mark>Now How the attacker will going to send an ambiguous request?</mark><br> 
 `Answer:` an attacker will attach the `malicious prefix(Orange-block)` with the `Orignal(Blue-block)` of request, and then the ambiguous request will first reaches to the Front-end server.
 + `Front-end` will thinks that this `Blue + Orange` block of data is one request, so immediately it will send the whole request to Back-end server.
 + `Back-end` for some reason it'll thinks that this message will finishes with second blue block and therefore it thinks that orange bit of data is the start of the next request and it's just gonna wait for that second request to be finished until that request is completed.

  **~ `Question` - what's gonna complete that request?**

+ Well, it could be someone else sending a request to the application. So an attacker can apply `arbitary prefix/content` to someone else request via smuggling and That's the core primitive of this technique `[check Fig-3]`.

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/3.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 3. Attacker, victim, Front-end and Back-end</figcaption>
</figure>

#### **How do this request smuggling arise?**

+ Most of the `request smuggling` vulnerabilities arise due to the HTTP specification provides two different ways to specify where a request ends:
1.) Content-Length header [CL]
2.) Transfer-Encoding header [TE]

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/4.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 4. Content-Length and Transfer-Encoding herders</figcaption>
</figure>

#### `Example:`
- `Content-Length header:`  it specifies the length of the message body in bytes.<br>see the `[Fig 4.]` as there are `six` characters in the POST body therefore the Content-Length header value is six.
- `Transfer-Encoding:` it's useful when we don't know the length of client request and response received from the server.<br>

<mark>Also it's used to specify that the message body uses chunked encoding, that means the message body contains one or more chunks of data.Each chunk consists of the chunk size in bytes (expressed in hexadecimal), followed by a newline, followed by the chunk contents. The message is terminated with a chunk of size zero.</mark><br>

```bash
Fig 4: Contains total three chunks of different size in bytes(hexadecimal), 
       followed by a CRLF(\r\n), followed by chunk data/contents.

  5\r\n     - chunk size in HEX followed by CRLF as line separator.
  Nihar\r\n - First chunk data of five characters in decimal   
              and obviously the 5 is the hexadecimal chunk size
              of the Nihar(chunk data).
---------------------------------------------------------------------------
  7\r\n       - chunk size in HEX followed by CRLF as line seperator.
  Rathod \r\n - Second chunk data of seven characters in
                decimal so the 7 is the hexadecimal chunk size
                of the (Rathod ) inclding one "space" after
                Rathod.
----------------------------------------------------------------------------
  10\r\n    - chunk size in HEX followed by CRLF as a line seperator
  is \r\n   - Third chunk -> 5 decimal including white space in first row.
  \r\n      - count as seperate 2 decimal(\r\n)
  Nachiket.\r\n - count as 9 decimal including period(.) sign
  0\r\n - The message will terminate with chunk size 0. total decimal number 16 = 10 hexadecimal.
  \r\n
  Note: Transfer-Encoding will stop reading after the terminating chunk size 0. that is why the last character was the period(.) sign.
```       
<br>

```text
Note: 
+ Burp Suite automatically unpacks chunked encoding to make messages easier to view and edit.
+ Browsers do not normally use chunked encoding in requests, and it is normally seen only in server responses
```

- <mark>Now as we know that HTTP specification provides two different methods(CL and TE) for specifying the length of HTTP messages right!.</mark><br>
- <mark>So now it might be possible for a single message to use the both methods(CL & TE) at same time, such that they will conflict with each other.</mark><br>
- <mark>The HTTP specification will prevent this conflict problem by stating that if both the Content-Length and Transfer-Encoding headers are present, then the Content-Length header should be ignored.</mark>
- <mark>This might be sufficent to avoid the ambiguity when only a single server in play, but not when two or more servers are chained together. In this situation, problems can arise for two reasons: 

```text
Note:
+ Some servers do not support the Transfer-Encoding header in requests.
+ Some servers that do support the Transfer-Encoding header can be induced not to process it if the header is obfuscated in some way.
```

+ So we can say that request smuggling vulnerabilities arise if the `Front-end` and `Back-End` servers behave differently in relation to the **Transfer-Encoding header**, then they might <mark>disagree about the boundries between successive requests</mark>, and will leads to request smuggling attack.


#### **How we can perform an HTTP request smuggling attack?**

+ As we know that this attack involves both `Content-Length` and `Transfer-Encoding` headers into a single request right!
+ So by **manipulating** the request so that the `Front-end` and `Back-End` servers process the request differently.

#### **Let's understand the simple approaches**

```
There are four basic approaches by which we can check whether the website is vulnerable with request smuggling or not?

1.) CL.CL: Both Front-end and Back-end server uses the Content-Length header.
2.) CL.TE: the front-end server uses the Content-Length header and the back-end server uses the Transfer-Encoding header.
3.) TE.CL: the front-end server uses the Transfer-Encoding header and the back-end server uses the Content-Length header.
4.) TE.TE: the front-end and back-end servers both support the Transfer-Encoding header, but one of the servers can be induced not to process it by obfuscating the header in some way. 
```

**1. Desynchronizing: the classic approach `CL.CL`**

<figure style="text-align: center;">
<img src="/assets/blogs/Smuggling/5.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 5. Front-end and Back-end [CL.CL]</figcaption>
</figure>

`Fig 5.` is an example of an ambiguous request, as we are using absolute classic old school Desynchronization technique.

+ In this example, we simply specifed Content-Length header (CL) twice.
+ Front-end will use `CL - 6` --> will forward data up to Orange one (12345A) to the Back-end.
+ Back-end will use `C.L - 5` --> and it'll thik that `Orange - A` is the start of the next request.

In above example, the injected `A` will corrupt the green user's real request and they will probably get a response along the lines of "Unknown method APOST".

`Note:` 
Above technique is old-school and classic that it doesn't actually work on anything that's worth hacking these days.

**2. Desynchronizing: the chunked approach `CL.TE`**

<figure style="text-align: center;">
<img src="/assets/blogs/Smuggling/6.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 6. Front-end and Back-end [CL.TE]</figcaption>
</figure><br>

In `Fig 6.` **Front-end** will check `CL` and **Back-end** will check the `TE` header. we can perform the simple `HTTP request smuggling` attack as follow:

+ Here Front-end will check the `Content-Length` which is **13**, so it will read the request up to the thirteen characters starting from `0 to the end SMUGGLED`.
+ After that the request will go to the Back-end.
+ Now **Back-end** will start reading from the <mark>first chunk size</mark> which is stated to be `0` over here. so obviously it's gonna terminate the further request from there.
+ So the word `SMUGGLED` is going to remain unprocessed over there until the next victim request will arrived.
+ Once the victim request will arrived over there will get a response **Unknown method SMUGGLEDPOST**.

**3. Desynchronizing: the `TE.CL` approach**

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/7.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 7. Front-end and Back-end [TE.CL]</figcaption>
</figure><br>

In `Fig 7.` **Front-end** will check `TE` and **Back-end** will check the `CL` header.

```text
Note:
+ To send this request using Burp Repeater, you will first need to go to the Repeater menu and ensure that the "Update Content-Length" option is unchecked.
```

+ Here the `Front-end` server processes the `Transfer-Encoding header`, so it will treat the entire message body as **chunked encoding**.
+ Now it will process the **first chunk**, which is stated to be **8 byte** long`(Fig 7.)` upto the  the start of the line following SMUGGLED.
+ Now it will process the **second chunk**, which is stated to be `zero length`, and so is treated as <mark>terminating the request.</mark> 
+ At last This request is forwarded on to the back-end server.
+ <mark>The back-end server processes the Content-Length header and determines that the request body is 3 bytes long, upto the start of the line following 8(including \r\n).</mark>
+ That means the following bytes, starting with the `SMUGGLED` are left unprocessed and back-end server will treat these chunk data as start of the next request in the sequence.

```text
Note:
+ This technique(TE.CL) works on quite a few systems, but we can exploit many more by making the TransferEncoding header slightly harder to spot, so that one system doesn't see it.
```

**4. Forcing Desync**

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/8.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 8. Forcing Desync quirks</figcaption>
</figure>

+ If a message is received with both a Transfer-Encoding header field and a ContentLength header field, the latter MUST be ignored. – RFC 2616 #

# 2. Methodology ⚛️
<hr>

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/9.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 9. Methodology</figcaption>
</figure>

+ The theory behind request smuggling is straightforward, but the number of uncontrolled variables and our total lack of visibility into what's happening behind the front-end can cause complications.

# 3. Detecting Desync 🕵️
<hr>

`IMP:`

+ <mark>To detect request smuggling vulnerabilities we've to issue an ambiguous request followed by a normal 'Victim' r equest, then observe whether the latter gets an unexpected response.</mark>
+ <mark>However, this is extremely prone to interference; if another user's request hits the poisoned socket before our victim request, they'll get the corrupted response and we won't spot the vulnerability.</mark>
+ <mark>This means that on a live site with a high volume of traffic it can be hard to prove request smuggling exists without exploiting numerous genuine users in the process.</mark>
+ <mark>Even on a site with no other traffic, you'll risk false negatives caused by application-level quirks terminating connections.</mark>

`So what will be the detecion strategy?`

+ Sequence of messages which make vulnerable backend systems hang and time out the connection.This technique has few `false positives`, and most importantly has virtually `no risk` of **affecting other users.**

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/10.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 10. Timing Techniques Example:1 and Example:2</figcaption>
</figure>

`Exammple-1:`<br>
1.) **CL.CL --> [Back-end Response]**

  + `Reason:` --> Front-end will check `CL` which is 6 so it will calculate the length upto the `Q` and forward the request to the Back-end.
  + once the request will reach to the back-end will again check for the CL which is 6 again and therefore again it's going to calculate upto the the last character `Q` and we'll get the normal response and every thing is fine.

2.) **TE.TE -->  [Front-end Response]**

  + `Reason:` --> Fornt-end will check the Transfer-Encoding and therefore will first check the chunk size which is stated to be **3** over here so it will read the next line chunk data which are `abc`.
  + Then it will check for the next chunk size for `Q` which is not defied and also terminating chunk size `0` is not  defied in the example-1 right? so that is why will get some error in resonse and Front-end will respond.

3.) **TE.CL --> [Front-end response]**

  + `Reason:` --> Same as [TE.TE]

4.) **CL.TE --> [Timeout]**

  + `Reason:` --> if you see the example 1 with this technique
  then Front-end server uses the `CL` header, so it will only forward the `Blue` part of data to the back-end and will ommit the `Q`.
  + Back-end server uses the `TE` header, so it will processes the first chunk and waits for the next chunk to arrive. This will cause an observable time delay.

`Example-2:`<br>
1.) **CL.CL --> [Back-end response]**
    
  + `Reason:` --> Check the Ex-2(on Right-side), we can see that Fornt-end is checking a `CL` header which is 6 bytes in length.
  ```
  0\r\n --> first 3 characters
  \r\n  --> \r\n in it's own seperate line
  x     --> last chuck data x
  ```
  + So it will send the whole request to the back-end upto the last `orange` character `x`.
  + Now Back-end will check the `CL` again and gives the normal response and everything is fine.

2.) **TE.TE --> [Back-end Response]**

  + `Reason:` --> Here, the Front-end is checking a `TE` header so it will process the first chunk which is `0` over here and therefore it will terminate the further request and send it to the Back-end.
  + back-end will check the `TE` header again and same as Front-end will stop reading after the terminating chunk size `0` and will get the normal response and everything is fine.

3.) **TE.CL --> [Timeout]**

  + `Reason:` --> Front-end server will use the `TE` header and will forward the `blue` part of data to the Back-end server(due to the terminating chunk size 0), and will ommit the `x`.
  + Back-end uses the  `CL` header and will expects more content in the message body, and due to that it's just going to waits for the remaining content to arrive. This will cause an  observable time delay.

  ```text
  Note:
  + The timing-based test for TE.CL vulnerabilities will potentially disrupt other application users if the application is vulnerable to the CL.TE variant of the vulnerability. So to be stealthy and minimize disruption, you should use the CL.TE test first and continue to the TE.CL test only if the first test is unsuccessful.
  ```

4.) **CL.TE --> [Socket poision ☠️]**

  + `Reason:` Front-end server will use the `CL` header and will forward the whole request including the last character `x` to the Back-end.
  + Now what happens is back-end will processes with the first chunk size which is stated `0` over here and therfore the remaining part of request(`x`)is going to remain unprocessed over there, and that is how we can poisoned the socket.

  ```text
  Note:
  This approach will poison the backend socket with an X, potentially harming legitimate users. Fortunately, by always running the prior detection method first, we can rule out that possibility.
  ```

# 4. Confirming desync 👍
<hr>

```text
+ In this step will see the full potential of request smuggling is to prove backend socket poisoning is possible.

+ To do this we'll issue a request designed to poison a backend socket, followed by a request which will hopefully fall victim to the poison.

+ If the first request causes an error the backend server may decide to close the connection, discarding the poisoned buffer and breaking the attack.

+ Try to avoid this by targeting an endpoint that is designed to accept a POST request, and preserving any expected GET/POST parameters.
```
`Note`:
Some sites have multiple distinct backend systems, with the front-end looking at each request's method,URL, and headers to decide where to route it. If the victim request gets routed to a different back-end from the attack request, the attack will fail. As such, the 'attack' and 'victim' requests should initially be as similar as possible.

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/11.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 11. Confirming Desync</figcaption>
</figure>

+ `[Fig 11.]` indicates two different methods.and that is how we can perform and confirm the smuggling attack.
+ `CL.TE` --> If the attack is successfull the victim request in green will get a `404` response.
+ `TE.CL` --> The TE.CL attack looks similar, but the need for a closing chunk means we need to specify all the headers ourselves and place the victim request in the body. Ensure the Content-Length in the prefix is slightly larger than the body.

```text
Note:
+ If the site is live, another user's request may hit the poisoned socket before yours, which will make your attack fail and potentially upset the user. As a result this process often takes a few attempts, and on hightraffic sites may require thousands of attempts. Please exercise both caution and restraint, and target staging servers were possible.
```
<br>

# 5. Explore 👽
<hr>

+ Application server validate http request length on the basis of two headers.
  1.) Transfer-Encoding
  2.) Content-Length

+ On Live scenario server has multiple load balancer or Frontend and Backend server which process the request. We are aim to exploit improper validation of request on application. Assume, We have 4 different scenarios:

1.) Frontend server is validating the request length via Transfer-Encoding and Backend server validating via Content-Length headers.
2.) Frontend server is validating the request length via Content-Length and Backend server validating via Transfer-Encoding headers.
3.) Frontend server is validating the request length via Content-Length and Backend server validating via Content-Length headers.
4.) Frontend server is validating the request length via Transfer-Encoding and Backend server validating via Transfer-Encoding headers.

## Live Demo:

<figure style="text-align: center;">
  <img src="/assets/blogs/Smuggling/12.gif" alt="picture" style="border:1px solid white"/>
<figcaption style="color: red;">Fig 12. HTTP request smuggling</figcaption>
</figure><br>

```shell
GET / HTTP/1.1
Host: 192.168.0.109
Content-Length: 4
Transfer-Encoding: chunked

2c\r\n
GET /path HTTP/1.1\r\n
Host: 127.0.0.1:8080\r\n
\r\n
\r\n
0
```
<br>

**On above example we are having the TE-CL Vulnerability on server. Let me explain all values one by one.**

+ **"Content-Length"** header in request is set according to the size of the `"2c\r\n"` bytes.
+ According to method, we are calculating the total size of first line of the content.
+ Here we also calculating the `"\r\n"` new line feed.
+ `"Transfer-Encoding"` header is calculated by total bytes of the content.
+ Here we are having simple HTTP GET request which size is `44` till the header ends, after `"\r\n\r\n 0"` which indicate to stop.
+ Decimal `44` is now converted to hexadecimal which gives `"2c"`. The reason we have added `"2c"` before the content is the total hexadecimal value of the content.
+ After the `"0"` we have to add two `"\r\n"` line feed and send the request to the server.
<br>

**If you send below request to the CTF server. which gives the response with the flag.**


```bash
GET /a HTTP/1.1
Host: 192.168.0.109
Content-Length: 4
Transfer-Encoding: chunkedasd

2c
GET /flag HTTP/1.1
Host: 127.0.0.1:8080


0

GET /a HTTP/1.1
Host: 127.0.0.1:8080
```

**- [LAB - HTTP request smuggling](https://www.vulnhub.com/entry/finithicdeo-1,636/)**

## References:

- [medium](https://medium.com/@knownsec404team/protocol-layer-attack-http-request-smuggling-cc654535b6f)
- [cgisecurity](https://www.cgisecurity.com/lib/HTTP-Request-Smuggling.pdf)
- [blackhat](https://i.blackhat.com/USA-19/Wednesday/us-19-Kettle-HTTP-Desync-Attacks-Smashing-Into-The-Cell-Next-Door.pdf)
- [YouTube](https://www.youtube.com/watch?v=_A04msdplXs&t=904s)