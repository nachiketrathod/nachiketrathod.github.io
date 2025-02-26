---
title: "Intercepting Desktop Application Via Fiddler"
tags:
    - Thick Client Security
    - Desktop Application
    - Fiddler
    - BurpSuite
date: "2023-04-03"
thumbnail: "/assets/img/thumbnail/thick.gif"
---

# About the blog
---
* Author: [Nachiket Rathod](https://www.linkedin.com/in/nachiketrathod/)
* Genre: `Thick Client`
* Objective: Ensuring the security and integrity of thick client applications by identifying vulnerabilities in desktop software that communicate with remote servers.

<h3 id="tldr"> 
     <strong>GT;DR</strong>
</h3>

Thick client application security testing is a crucial process for ensuring the `security` and `integrity` of software applications that run on end-user machines. These **"fat" or "rich"** client applications often have a GUI that communicates with a remote server or database, making them vulnerable to a range of security threats.  `Desktop applications` are software programs that are installed on local computers or workstations. They are different from web applications, which run over a network or the internet. Examples of desktop applications include word processors, image editors, and financial software, among others. While desktop applications have many benefits, such as faster performance and greater control over system resources, they are also susceptible to security risks.


# Challenges intercepting Windows app traffic 👻

When you download a desktop application from `Windows Store`, it may be **difficult** to intercept its traffic in tools like `Burp Suite` or `Echo Mirage`, depending on how the application is designed and how it communicates with servers.

It may be difficult to intercept traffic from a desktop application is that it may use a `different protocol` or `communication method` than traditional web applications. For example, it may communicate over a `custom protocol`, use a `proprietary encryption` method, or use `binary data` formats that are not easily readable by tools like Burp Suite or Echo Mirage.

**`Additionally`**, some desktop applications may be designed to prevent interception of their traffic for security or anti-piracy reasons. They may use techniques like `SSL pinning`, which prevents the interception of SSL traffic, or `code obfuscation`, which makes it difficult to reverse-engineer the application.

# Overcoming Traffic Intercept Obstacles🪓

In our case, We analyzed the traffic of a **`desktop application`** downloaded from the `Windows Store` using Wireshark. During our analysis, we observed TCP traffic, but we encountered issues when attempting to intercept it using Echo Mirage, as the **application crashed**. We also attempted to intercept the traffic by configuring the **system proxy** with **Burp Suite**, but were unsuccessful. The **`invisible proxy`** method was also unsuccessful in our attempts to intercept the traffic.

We then started searching other methods for `intercepting traffic` from the application. We went through many blogs and forums, but did not find the right solution. Finally, we came across a solution on the internet that suggested using Fiddler to capture network traffic between the internet and test computers.

## **`Fiddler`**

`Fiddler` is a powerful **`proxy server`** that can capture traffic from any application that supports `HTTP or HTTPS` traffic. By setting Fiddler as the **system proxy**, we were able to intercept traffic from the desktop application without any issues. We could see the requests and responses in clear text, and we could also manipulate the traffic using Fiddler's built-in tools.

# Pre-Requisites:

1. [Fiddler](https://www.telerik.com/download/fiddler)
2. [Burp Suite](https://portswigger.net/burp/communitydownload)
3. [Your Application from windows store](https://www.microsoft.com/en-ww/store/)


# Let’s Get Started:

1.) First install the applicaton from the Windows Store.

<figure  style="text-align: center;">
<img src="/assets/blogs/ThickClient/1.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 1. Application installed from Windows Store.</figcaption>
</figure><br>

2.) Install the [Fiddler](https://www.telerik.com/download/fiddler) from the site.

<figure  style="text-align: center;">
<img src="/assets/blogs/ThickClient/2.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 2. Fiddler is now installed and ready to use.</figcaption>
</figure><br>

3.) Now configure the Fiddler to intercept the traffic from the Windows applicaiton.

<figure  style="text-align: center;">
<img src="/assets/blogs/ThickClient/3.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 3. Click on the WinConfig, Select your applicaiton and click on Save Changes.</figcaption>
</figure><br>

4.) Now enable capturing and launch your applicaiton.

<figure  style="text-align: center;">
<img src="/assets/blogs/ThickClient/4.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 4. Observe the HTTP/HTTPS traffic from the targated application.</figcaption>
</figure><br>

5.) Now, Go to the **`Tools`** --> **`Gateway`** --> **`Manual Proxy`** Configuration, and setup the proxy. Also setup the same way in Burp Suite(All interfaces).

<figure  style="text-align: center;">
<img src="/assets/blogs/ThickClient/5.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 5. Click on ok and Save the Settings .</figcaption>
</figure><br>

6.) Now, agin launch the application, try to intercept the traffic in Fiddler and observe that we can now be able to see the traffic in Burp as well.

<figure  style="text-align: center;">
<img src="/assets/blogs/ThickClient/6.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 6. Observe that traffic has been successfully intercepted in the Burp Suite.</figcaption>
</figure><br>


## Note:

When fiddler ask for install the certificates accept the necessary ones and it will automatically set-up the rest. some of your enterprise applications might notice the proxy change and stop working, but at least you can get through your test.

### Ally: 

~ [Suresh budarapu](https://www.linkedin.com/in/suresh-budarapu-74b5463b)