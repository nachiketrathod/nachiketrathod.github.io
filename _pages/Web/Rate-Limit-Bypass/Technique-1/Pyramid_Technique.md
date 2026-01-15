---
title: "Rate-Limit Evasion Through Pyramid Technique"
tags:
    - Rate-Limit
    - Login
    - Bruteforce
date: "2023-11-07"
thumbnail: "/assets/img/thumbnail/ratelimit.gif"
---

# About the blog
---
* Author: [Suresh Budarapu](https://www.linkedin.com/in/sureshbudarapu/)
* Genre: `Web Application`
* Objective: Bypassing a robust login rate-limiting mechanism through perseverance and innovative problem-solving.

<h3 id="gtdr"> 
     <strong>GT;DR: </strong>
</h3>

In a recent assessment of a web application's security, a **`robust login`** **rate-limiting** mechanism was encountered. After several unsuccessful login attempts, the application blocked further access and displayed an error message. Despite this challenge, we explored various methods to bypass the mechanism. Initially, `standard techniques` didn't work, but with determination and **`creative problem-solving`**, we successfully bypassed the rate-limiting mechanism. This experience emphasized the importance of perseverance and innovative thinking in overcoming complex security challenges.

# Challanges 🔒
<!--#### **What is HTTP Request Smuggling?**
<mark>It's a technique for interfering with the way of website process the sequences of HTTP requests that are received from one or more users.</mark>-->
We've tried the all possible known methods but no luck!

 ```
 ~ Attampted Bypasses
 ```

+ Using **`Special Characters`** ❌

  ``` text
    Null Byte (%00) at the end of the email.
    Common characters that help bypassing the rate limit: 
    0d, %2e, %09, %0, %00, %0d%0a, %0a, %0C.
  ```

+ Adding **`HTTP Headers & IP Spoof`**: ❌
 ```sass
    X-Forwarded-For: IP
    X-Forwarded-IP: IP
    X-Client-IP: IP
    X-Remote-IP: IP
    X-Originating-IP: IP
    X-Host: IP
    X-Client: IP
    X-Forwarded: 127.0.0.1
    X-Forwarded-By: 127.0.0.1
    X-Forwarded-For: 127.0.0.1
    X-Forwarded-For-Original: 127.0.0.1
    X-Forwarder-For: 127.0.0.1
    X-Forward-For: 127.0.0.1
    Forwarded-For: 127.0.0.1
    Forwarded-For-Ip: 127.0.0.1
    X-Custom-IP-Authorization: 127.0.0.1
    X-Originating-IP: 127.0.0.1
    X-Remote-IP: 127.0.0.1
    X-Remote-Addr: 127.0.0.1
  ```

## Investigating Rate Limiting Mechanism: 🕵️‍♂️

After trying **known methods**, we attempted to understand how the rate limiting was implemented. We intercepted the requests and tried to brute force using an intruder. We tested **`180 different password combinations`**. After completing the attack, we 	observed that the application implemented rate limiting after **`23 unsuccessful attempts`**, throwing the **error message:** 	<mark>'Too many incorrect attempts. Please try again later.'</mark> This occurred even when entering a valid username.

## Demonstrating the Attack: 🪓

**Step-1:**
 * Intercept the application's `login` request and redirect it to the intruder. 
 * Next, select the password position and choose the **`sniper-attack`** option. 
 * Enter the **`password combinations`** and initiate the attack.
 *  Upon completion of the attack, observe that the application threw an `error message` after multiple incorrect attempts, indicating the **presence of proper rate limiting**

<figure  style="text-align: center;">
<img src="/assets/blogs/RateLimit/Technique-1/1.png" alt="F1" style="border:2px solid purple">
<figcaption style="color: red;">Fig 1. The application has implemented a request rate limit.</figcaption>
</figure>

**Step-2:**
* After completing the attack, attempt to enter the `valid password` in the **repeat request**, resulting in an error.
* Despite entering the **`correct password`**, the error still persists.

<figure  style="text-align: center;">
<img src="/assets/blogs/RateLimit/Technique-1/2.png" alt="F2" style="border:2px solid purple">
<figcaption style="color: red;">Fig 2. Observe the response with valid passowrd</figcaption>
</figure>

**Step-3:**
* Now, append the common character **`'%20'`** to the username parameter.
* Surprisingly, this **bypassed the rate-limiting** and allowed logging into the application using the valid password even after `numerous incorrect password` attempts.

<figure  style="text-align: center;">
<img src="/assets/blogs/RateLimit/Technique-1/3.png" alt="F3" style="border:2px solid purple">
<figcaption style="color: red;">Fig 3. observe the success response(Redirection).</figcaption>
</figure>

**Step-4:**
* Permform the brute-force attack to find the valid password **after adding `'%20'` to the username** parameter.
* Even after completing the brute force attack, the `error message` remained the same: <mark>'Too many incorrect attempts. Please try again later.'</mark>
* Unfortunately, `adding` the command character **`'%20' didn't work during the brute-force`** attack.

<figure  style="text-align: center;">
<img src="/assets/blogs/RateLimit/Technique-1/4.png" alt="F4" style="border:2px solid purple">
<figcaption style="color: red;">Fig 4. Observe the error message again.</figcaption>
</figure>

**Step-5:**
* Upon attempting a brute force attack, repeat **`Step-2`** and observe the same error message.

<figure  style="text-align: center;">
<img src="/assets/blogs/RateLimit/Technique-1/5.png" alt="F5" style="border:2px solid purple">
<figcaption style="color: red;">Fig 5. Observe the same error message.</figcaption>
</figure>

**Step-6:**
* After a comprehensive analysis, a **`new method`** was attempted: **`incrementally adding '%20'`** in each request, which resulted in `bypassing the rate-limiting` of password attempts.
* For **`example`**, in the first request, **Add** the **`'%20' once`**, and in the **`second request, add it twice`**. Repeat this process till 180 requests, adding 180 '%20' characters.
* Choose the Attack-type: **`Pitchfork`**.

<figure  style="text-align: center;">
<img src="/assets/blogs/RateLimit/Technique-1/6.png" alt="F6" style="border:2px solid purple">
<figcaption style="color: red;">Fig 6. Choose the attack type and payload positions.</figcaption>
</figure>

**Step-7:**
* Utilize below `Python code` to generate incremental '%20' characters.

```bash
# Initialize the initial string
pattern = "%20"
 
# Loop 180 times to print the pattern
for i in range(1, 181):
    # Print the pattern for the current iteration
    print(f"Time {i}: {pattern}")
    # Add another "%20" to the pattern for the next iteration
    pattern += "%20"
```

* Utilize the generated payloads at **`position one`**, which corresponds to the `username parameter`.

<figure  style="text-align: center;">
<img src="/assets/blogs/RateLimit/Technique-1/7.png" alt="F7" style="border:1px solid purple">
<figcaption style="color: red;">Fig 7. Add the output as payloads.</figcaption>
</figure>

**Step-8:**
* Observe the `last request` after completing the attack, where **`'%20' was appended`** 180 times to the username parameter.

<figure  style="text-align: center;">
<img src="/assets/blogs/RateLimit/Technique-1/8.png" alt="F8" style="border:2px solid purple">
<figcaption style="color: red;">Fig 8. observe the  heighted request.</figcaption>
</figure>

**Step-9:**
* Observe that the login **`rate-limit`** was **`bypassed`**.
* Allowing the application to find a **`valid credentials`** after numerous unsuccessful password attempts.

<figure  style="text-align: center;">
<img src="/assets/blogs/RateLimit/Technique-1/9.png" alt="F10" style="border:2px solid purple">
<figcaption style="color: red;">Fig 9. Observe the success response</figcaption>
</figure>

# Conclusion:

It's crucial **`not to give up`** when faced with login rate limiting on applications. If default methods prove ineffective, there are **`numerous alternative ways`** to bypass these limitations. By thinking `outside the box` and creating new methods, it's possible to overcome even the most robust security measures. This adaptability underscores the importance of continuous **`vigilance and innovation`** in the field of security.

## Ally:

+ [Suresh Budarapu](https://www.linkedin.com/in/suresh-budarapu-74b5463b/)
