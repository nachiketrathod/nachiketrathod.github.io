---
title: "Intercepting Flutter Android(ARMv8) Application"
tags:
    - Flutter
    - Android
    - SSL Pinning Bypass
    - ghidra
    - Frida
    - Proxydroid
date: "2021-06-23"
thumbnail: "/assets/img/thumbnail/flutter.png"
---

# About the blog
---
* Author: [Nachiket Rathod](https://www.linkedin.com/in/nachiketrathod/)
* Genre: `Mobile Application`
* Objective: Intercepting and analyzing the network traffic of Flutter-based Android applications using Frida, Ghidra, and Proxy Droid for dynamic analysis.

<h3 id="tldr"> 
     <strong>TL;DR: </strong>
</h3>

In this blog we'll see how to intercept the traffic of [`Flutter`](https://flutter.dev/) based `Android` application for dynamic analysis with the help of `Frida`, `ghidra` and `Proxy droid`.

# Introduction

As Flutter uses dart, Dart is not proxy aware and uses its own certificate store. Hence, The application doesn’t take any proxy settings from the system and sends data directly to server, because of this we cannot intercept the request using [`Burpsuite`](https://portswigger.net/burp) or any MITM tool, so changing the proxy settings in wifi or trusting any certificate won’t help here.

**~ Question what we can do in this condition?**<br>

+ Here comes [`ProxyDroid`](https://play.google.com/store/apps/details?id=org.proxydroid&hl=en_IN&gl=US) in picture which modifies iptables in android system to send the traffic to burp proxy.

# Pre-Requisites:

1. [Rooted android device](https://www.xda-developers.com/root/)
2. [Burp Suite](https://portswigger.net/burp)
3. [ProxyDroid](https://play.google.com/store/apps/details?id=org.proxydroid&hl=en_IN&gl=US)
4. [Frida client and server](https://frida.re/)
5. [Ghidra/ida-pro](https://ghidra-sre.org/)
6. [Droid Hardware Info](https://play.google.com/store/apps/details?id=com.inkwired.droidinfo&hl=en_IN&gl=US)
7. Your system and mobile device must be connected to same Wi-Fi network.

# Let’s Get Started:

1.) First install the `Droid Hardware Info` to the Android device from Play store and check the ARM version of your android device in my case it's [`ARMv8`](https://en.wikipedia.org/wiki/ARM_architecture).

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/1.png" alt="droidinfo" style="border:2px solid purple; width: 50%; height: auto;">
<figcaption style="color: red;">Fig 1. End users, Front-end and Back-end.</figcaption>
</figure>

    Note:

    + As flutter has it own engine to run application, there are different libraries developed based on CPU architectures, In order to find the CPU architecture we need  droid info app to get the CPU architecture details
    Check the details on Instruction set to see the CPU architecture.
 
 2.) Now after the archicture verification install the `ProxyDroid` from the playstore.

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/2.jpg" alt="picture" alt="droidinfo" style="border:2px solid purple; width: 50%; height: auto;">
<figcaption style="color: red;">Fig 2. End users, Front-end and Back-end.</figcaption>
</figure><br>


    ~ Setup the Following:
    1. HOST: Burp Proxy hostname
    2. Proxy: burp proxy port
    3. Proxy Type: HTTP
    4. Leave the remaining settings unchanged.

3.) Download the `flutter` based APK to your PC and extract the apk via 7-zip or just decompile the apk via [`apktool`](https://installlion.com/kali/kali/main/a/apktool/install/index.html) [`command: apktool d APK filename`].

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/3.png" alt="picture" alt="droidinfo" style="border:2px solid purple">
<figcaption style="color: red;">Fig 3. End users, Front-end and Back-end.</figcaption>
</figure><br>

4.) After extracting the apk file, go to the `lib` folder.

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/4.png" alt="picture" alt="droidinfo" style="border:2px solid purple">
<figcaption style="color: red;">Fig 4. End users, Front-end and Back-end.</figcaption>
</figure><br>

5.) Now select the desired archicture that we have taken from **droid hardware info** app (arm64-v8). Now in your case these archicture folders may resides inside the `bin` folder.

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/5.png" alt="picture" alt="droidinfo" style="border:2px solid purple">
<figcaption style="color: red;">Fig 5. End users, Front-end and Back-end.</figcaption>
</figure><br>

6.) Once we move into desired CPU architecture folder(e.g here **arm64v8**) we can see the **libflutter.so**. This is the file that contains all the `functions` and `libraries` to call SSL pinning in the app according to CPU archicture.


<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/6.png" alt="picture" alt="droidinfo" style="border:2px solid purple">
<figcaption style="color: red;">Fig 6. End users, Front-end and Back-end.</figcaption>
</figure><br>

7.) Set the Host to system Ip and port 8081 which is set as in `ProxyDroid`. also Identified SSL verification implemented using x509.cc. Run the application installed on the android device to capture the traffic and observe the burpsuite dashboard -> Event logs. We got TLS handshake error, Hence we can not intercept https traffic.

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/7.png" alt="picture" alt="droidinfo" style="border:2px solid purple">
<figcaption style="color: red;">Fig 7. End users, Front-end and Back-end.</figcaption>
</figure><br>

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/8.png" alt="picture" alt="droidinfo" style="border:2px solid purple">
<figcaption style="color: red;">Fig 8. End users, Front-end and Back-end.</figcaption>
</figure><br>

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/9.png" alt="picture" alt="droidinfo" style="border:2px solid purple">
<figcaption style="color: red;">Fig 9. End users, Front-end and Back-end.</figcaption>
</figure><br>

8.) Install Ghidra and dissemble the `libflutter.so` file.
file path C:\Desktop\flutter\yourfolder\lib\arm64-v8a
Start Ghidra and drag and drop the Flutter file to it.

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/10.png" alt="picture" alt="droidinfo" style="border:2px solid purple">
<figcaption style="color: red;">Fig 10. End users, Front-end and Back-end.</figcaption>
</figure><br>

    
+ Once libflutter.so is imported in ghidra, select all default details. analyse the fucntions and packages.This analysis process take time so don't worry about that(5-10mins approx.)

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/11.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
<figcaption style="color: red;">Fig 11. End users, Front-end and Back-end.</figcaption>
</figure><br>

+ Now Once Analysis is completed search for the `strings` option.

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/12.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
<figcaption style="color: red;">Fig 12. End users, Front-end and Back-end.</figcaption>
</figure><br>

+ Search for [`x509.cc`](https://github.com/google/boringssl/blob/master/ssl/ssl_x509.cc#L362) string in the binary, and we can see, we have got four XREF against the line. It’s pretty obvious that the ssl_x509.cc class has been compiled somewhere in the `0x650000` region, but that’s still a lot of functions to try to find the correct one. If searching for the filename doesn’t work, maybe searching for the line number would work.

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/13.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
<figcaption style="color: red;">Fig 13. End users, Front-end and Back-end.</figcaption>
</figure><br>

+ Also perform the `Scalar search` for Magic number `0x186` To understand what is this magic number in details I suggest you should read this blog by nviso Team.

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/14.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
<figcaption style="color: red;">Fig 14. End users, Front-end and Back-end.</figcaption>
</figure><br>

+ Map scalar search near to the x509.cc output and observe that one address we found in scalar search is in the range of the XREF[4] we got for x509.cc.

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/15.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
<figcaption style="color: red;">Fig 15. End users, Front-end and Back-end.</figcaption>
</figure><br>

+ now click on the following three heighlited parts. as we can see there so many `Locations` we have found in the scalar search but we choose the `0x65000` region because it has XREF[4] in [`x509.cc`].<br>

```text
Note: check the  XREF[4]  with step 11 
```

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/16.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
<figcaption style="color: red;">Fig 16. End users, Front-end and Back-end.</figcaption>
</figure><br>

+ Click on that address range `FUN_0065a4ec` and Observe the initial bytes value of `FUN_0065a4ec`. Copy some of those bytes:

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/17.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
<figcaption style="color: red;">Fig 17. End users, Front-end and Back-end.</figcaption>
</figure><br>

+ Copy the initial bytes as below and send to binwalk for offset count if needed.
  
```
ff 03 05 d1 fc 6b 0f a9 f9 63 10 a9 f7 5b 11 a9 f5 53 12 a9 f3 7b 13 a9 08 0a 80 52
```

```bash
# The first bytes of the FUN_0065a4ec function
ff 03 05 d1 fc 6b 0f a9 f9 63 10 a9 f7 5b 11 a9 f5 53 12 a9 f3 7b 13 a9 08 0a 80 52
# Find it using binwalk
binwalk -R "\xff\x03\x05\xd1\xfc\x6b\x0f\xa9\xf9\x63\x10\xa9\xf7\x5b\x11\xa9\xf5\x53\x12\xa9\xf3\x7b\x13\xa9\x08\x0a\x80\x52" libflutter.so
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
5612780       0x55A4EC        Raw signature (\xff\x03\x05\xd1\xfc\x6b\x0f\xa9\xf9\x63\x10\xa9\xf7\x5b\x11\xa9\xf5\x53\x12\xa9\xf3\x7b\x13\xa9\x08\x0a\x80\x52)

```

+ Replace the function value in the below Frida script as a pattern variable.

```js
function hook_ssl_verify_result(address)
{
  Interceptor.attach(address, {
    onEnter: function(args) {
      console.log("Disabling SSL validation")
    },
    onLeave: function(retval)
    {
      console.log("Retval: " + retval)
      retval.replace(0x1);
  
    }
  });
}
function disablePinning(){
    // Change the offset on the line below with the binwalk result
    // If you are on 32 bit, add 1 to the offset to indicate it is a THUMB function.
    // Otherwise, you will get  'Error: unable to intercept function at ......; please file a bug'
    var address = Module.findBaseAddress('libflutter.so').add(0x55a4ec)
    hook_ssl_verify_result(address);
}
setTimeout(disablePinning, 1000)
```

+ Connect mobile using USB with your system and run `adb shell` to find the package name of your apk.


<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/18.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
</figure>
<br>

+ Run frida script to your application package Name using below command:

```
> frida –Uf `package_name` -l `Bypass script` --no-pause
```
<br>
<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/19.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
</figure>
<br>
+ Perform some activity in the application and check back to burp to see the requests from app being intercepted now:

<figure  style="text-align: center;">
<img src="/assets/blogs/Flutter/20.png" alt="picture" alt="droidinfo" style="border:2px solid purple" width="1000">
</figure>
<br>

# Conclusion:

This blog was to share how I have bypassed the security implementation of an Android application, and how I have intercepted the traffic of flutter Android application. As the method for the same is different compare to what we actually do in mobile application testing to intercept the traffic.

# Reference:

[nviso](https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/)