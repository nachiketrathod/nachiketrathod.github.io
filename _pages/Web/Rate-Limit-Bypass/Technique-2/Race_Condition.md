---
title: "Rate-Limit Evasion Through Race Condition"
tags:
    - Rate-Limit
    - Login
    - Race Condition
    - Brute-force
date: "2025-04-16"
thumbnail: "/assets/img/thumbnail/tron.gif"
---

# About the bog
---
* Author: [Nachiket Rathod](https://www.linkedin.com/in/nachiketrathod/)
* Genre: `Web Application`
* Objective: Bypassing a robust login rate-limiting mechanism through Race Condition.

<h3 id="TLDR"> 
     <strong>TL;DR: Breaking the Limit </strong>
</h3>

Encountered a **`secure login`** **rate-limiting** mechanism that blocks access for a set duration after multiple failed login attempts. Standard bypass methods failed like trying to pick a **digital lock with a toothpick 😜** but with persistence and creative problem-solving approach, the rate limit was successfully bypassed, reinforcing the importance of perseverance in security testing. 🚀

# Challenges 🔐
We've tried the all possible known methods but no luck!

**Attempted Bypasses:**

+ Using **`Special Characters:`** ❌

  ``` text
    Null Byte (%00) at the end of the email.
    Common characters that help bypassing the rate limit: 
    0d, %2e, %09, %0, %00, %0d%0a, %0a, %0C, %20.
  ```

+ Adding **`HTTP Headers & IP Spoof:`** ❌
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

After exploring all possible **known methods**, an analysis was conducted to understand how the rate-limiting was implemented. Requests were intercepted, and a brute-force attempt was made using an intruder. Over **`100 different password combinations`** were tested. Upon completing the attack, it was observed that the application enforced rate limiting after **`three unsuccessful attempts`**, displaying the **error message:** <mark>"Too many incorrect attempts. Please try again later after 10 minutes."</mark>

## Race Condition: 🏃

Race conditions are a common security vulnerability related to business logic flaws. They happen when a `website` processes **multiple requests** at the same time without proper safeguards. This allows different `threads` to access and modify the same data simultaneously, leading to conflicts or unexpected behaviour. In a **race condition attack**, an attacker sends carefully timed requests to trigger these conflicts and manipulate the application for malicious purposes.

👉 The key challenge in exploiting race conditions is ensuring that `multiple requests` are processed `simultaneously` with minimal timing differences, ideally within **1 millisecond** or **less**.

 **HTTP/2: <mark>Single-Packet Attack</mark> `v/s.` HTTP/1.1: <mark>Last-Byte Synchronization</mark>**

👉 `HTTP/2:` Allows **two requests** to be sent over a **single TCP connection**, minimizing network jitter. However, due to server-side variations, relying on just two requests may not consistently trigger a race condition.

👉 `HTTP/1.1:` Last-Byte Sync: Enables **pre-sending** most of the data for `20-30 requests`, holding back a small portion, and then releasing it **simultaneously** to force synchronized processing on the server.

**<mark>Steps for Last-Byte Sync Preparation:</mark>**
1.) Send request **`headers and body`**, leaving out the final byte, while **keeping the stream open**.
2.) Pause for **`100ms`** after the initial send.
3.) Disable **`TCP_NODELAY`** to leverage Nagle’s algorithm, ensuring final frames are batched.
4.) Send a **ping** to warm up the connection.
5.) Release the **`withheld frames`**, ensuring they arrive in a `single packet` (verifiable via Wireshark).

**Note:** This method is ineffective for static files, as they are not typically involved in race condition exploits.

🔹**<mark> Adapting to Server Architecture:</mark>**
Understanding the target's architecture is key. **`Front-end servers`** may route requests differently, affecting timing. Sending harmless pre-requests can help normalize request timing by warming up **server-side** connections.

🔹**<mark> Handling Session-Based Locking:</mark>**
Some frameworks, like PHP’s session handler, **`serialize requests per session`**, making vulnerabilities harder to detect. Using unique session tokens for each request can bypass this restriction and improve attack timing.

🔹**<mark> Overcoming Rate or Resource Limits:</mark>**
If connection warming doesn’t work, intentionally flooding the server with dummy requests can trigger rate or resource limit delays. This can create conditions favorable for a **single-packet** attack, increasing the chances of a successful race condition exploit.

## The Attack: 🪓
<hr>

<h3 align="center">
  <span style="color: #ff7043;">
    🔭 The Manual Approach
  </span>
</h3>
**Step-1:** Intercept the application's `login` request and redirect it to the intruder, then send the request and observe the normal **request** and its **response** using valid `credentials`.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/1.png" alt="F1"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red; font-size: 1rem; margin-top: 10px;">
    Fig 1: Observe the valid request and response.
  </figcaption>
</figure><br>


**Step-2:** Send the **request** to the **Intruder**, select the password position, and choose the **`sniper-attack`** option. Enter the **`password combinations`**, including **one valid password** and the **rest as invalid**. Initiate the attack, and upon completion, observe the `remaining attempts` in the response.


<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/2.png" alt="F2"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 2: Observe the remaining attempts.</figcaption>
</figure><br>


**Step-3:** Navigate to the **third** request in **`Intruder`** and examine the response to check the remaining attempts.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/3.png" alt="F3"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 3: Observe the remaining attempts: Zero.</figcaption>
</figure><br>


**Step-4:** Same `step-3`, navigate to the **`100th`** request with a **valid password** and observe its response.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/4.png" alt="F4"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 4: Observe the remaining attempts: Zero.</figcaption>
</figure><br>

**Step-5:** Now, for the manual **Race-Condition** attack, create `100 tabs` in **Repeater**, with one **valid** password and the rest as **invalid**. Use Burp's **`Create Group`** feature to combine all the tabs into a **single group**.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/5.png" alt="F5"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 5: Screenshot shows the Create/Edit Group feature.</figcaption>
</figure><br>

**Step-6:** The first 99 tabs contain **incorrect passwords**, except for the last one. Now, select **`Send group in parallel (single-packet attack)`**, and then click the Send button to dispatch all requests in the group.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/6.png" alt="F6"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 6: Screenshot shows single-packet attack.</figcaption>
</figure><br>

**Step-7:** Observe the remaining attempts in the response. For example, in this case, the first tab shows `-40`.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/7.png" alt="F7"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 7: Screenshot shows the error message with 401 with remaining attempts.</figcaption>
</figure><br>

**Step-8:** Navigate to any **`other tabs`** and observe that the same `remaining attempts` value appears in the response. **For example**, in the screenshot below, the value is `-79`.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/8.png" alt="F8"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 8: Screenshot shows the error message with 401 with remaining attempts.</figcaption>
</figure><br>

**Step-9:** Navigate to the `last tab` and observe that the request containing the **`correct password`** received a valid response with a `user token`.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/9.png" alt="F9"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 9: Screenshot shows the success response.</figcaption>
</figure><br>


<hr>

<h3 align="center">
  <span style="color: #ff7043;">
    🔭 The Automated Approach 👉 Turbo Intruder
  </span>
</h3>


🔹This section demonstrates a method to exploit a **race condition** vulnerability in a login page by utilizing [Turbo Intruder](https://portswigger.net/research/turbo-intruder-embracing-the-billion-request-attack), an advanced HTTP request engine designed for high-speed and parallel attacks.
🔹The provided **Python script** uses Turbo Intruder’s request engine to perform a brute-force attack by sending multiple **login attempts in parallel**. These requests are queued and dispatched simultaneously through a single gate mechanism, leveraging a `single-packet attack` technique to bypass conventional rate limiting and account lockout protections.

```
def queueRequests(target, wordlists):
    # If the target supports HTTP/2, use Engine.BURP2 to trigger the single-packet attack
    # If they only support HTTP/1, use Engine.THREADED or Engine.BURP instead
    # For more information: https://portswigger.net/research/smashing-the-state-machine
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )

    # Assign a list of candidate payloads from the clipboard
    payloads = wordlists.clipboard  # Ensure you copy a wordlist to your clipboard

    # The 'gate' argument withholds part of each request until openGate is invoked
    # If you see a negative timestamp, the server responded before the request was complete
    for password in payloads:  # Iterating through password wordlist
        engine.queue(target.req % password, gate='race1')  # Injecting the payload into the request

    # Once every 'race1'-tagged request has been queued, send them in sync
    engine.openGate('race1')

def handleResponse(req, interesting):
    table.add(req)
```

### ⌛ Script 👉 Multiple Gates 
```
def queueRequests(target, wordlists):
    # Use HTTP/2 for a single-packet attack
    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=5,  # Allow parallel request queuing
        engine=Engine.BURP2
    )
    # Get passwords from clipboard
    passwords = wordlists.clipboard
    # Use multiple gates to avoid Turbo Intruder's 99-request limit
    max_per_gate = 50  # Adjust based on testing
    gate_index = 1
    current_gate = "race" + str(gate_index)  # Fixed string formatting
    count = 0
    for password in passwords:
        engine.queue(target.req % password, gate=current_gate)
        count += 1
        # If we reach the max_per_gate limit, switch to a new gate
        if count % max_per_gate == 0:
            engine.openGate(current_gate)  # Release current batch
            gate_index += 1
            current_gate = "race" + str(gate_index)  # Fixed formatting
    # Ensure the last batch is sent
    engine.openGate(current_gate)
def handleResponse(req, interesting):
    table.add(req)
```

🔹This approach targets systems vulnerable to **`TOCTOU (Time-of-Check to Time-of-Use)`** flaws, where concurrent requests may result in a successful login even if **traditional defenses** are in place. When one of the synchronized requests contains valid credentials, the **race condition** may allow `unauthorized access` before the system processes invalid attempts. This technique is particularly effective against applications that do not implement proper `concurrency control` during authentication checks.


**Step-10:** Copy the list of **passwords** from the `Notepad` file.
 

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/10.png" alt="F10"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 10: Screenshot showing the passwords copied to the clipboard.</figcaption>
</figure><br>


**Step-11:** Select the **`password value`** from the Repeater tab and send the request to **Turbo Intruder**. The selected password will **automatically** be replaced with `%s` as a placeholder for payload injection.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/11.png" alt="F11"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 11: Screenshot shows the python script for the single-packet attack.</figcaption>
</figure><br>

**Step-12:** Navigate to any `random Queue ID`(with Incorrect password) and observe the **"remaining attempts"** value in the response. Example: In the screenshot below, it shows `-21`.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/12.png" alt="F12"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 12: Screenshot showing the remaining attempts value in negative.</figcaption>
</figure><br>

**Step-13:** Navigate to any other Queue ID where the **`actual remaining attempts`** are reflected accurately. Example: In the screenshot below, it displays `0`.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/13.png" alt="F13"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 13: Screenshot shows the actual remaining attempts</figcaption>
</figure><br>

**Step-14:** Navigate to the last **`Queue ID (102)`** where the actual response is received. This confirms that the **brute-force** attack was successfully executed using `Turbo Intruder` through a **single-packet attack** by sending all requests in parallel.

<style>
  .hover-zoom {
    transition: transform 0.3s ease;
  }
  .hover-zoom:hover {
    transform: scale(1.05);
    z-index: 1;
  }
</style>

<figure style="text-align: center;">
  <img src="/assets/blogs/RateLimit/Technique-2/14.png" alt="F14"
       class="hover-zoom"
       style="border: 2px solid purple; width: 100%; max-width: none; height: auto;
              border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.4); image-rendering: auto;">
  <figcaption style="color: red;">Fig 14: Screenshot shows the successful single-packet attack.</figcaption>
</figure><br>


## ⚠️ Turbo Intruder Limitations & Fixes

### 1. **Internal Request Batching**
- Turbo Intruder appears to have an undocumented **batching threshold** (typically around `99–100` requests).
- Requests queued beyond this threshold may be **silently dropped**, even if `engine.openGate()` is used.

---

### 2. **`concurrentConnections=1` Causes Queuing Bottlenecks**
- With only a single connection, all requests are funneled sequentially.
- This may lead to:
  - ❌ **Delayed** request delivery  
  - ❌ **Dropped** requests  
  - ❌ **Improper synchronization**

---

### 3. **HTTP/2 Multiplexing Limits**
- Targets using HTTP/2 via `Engine.BURP2` may enforce **concurrent stream limits**.
- Exceeding those limits results in:
  - 🚫 **Stream resets**
  - 🚫 **Requests not processed or acknowledged**

---

### 4. **Single Gate Overload**
- Assigning all requests to the same gate (e.g., `race1`) can create internal processing issues.
- After about 99 requests:
  - ⚠️ The rest may be **ignored silently**
  - ⚠️ The attack becomes **incomplete or ineffective**

---

## Remediations and Best Practices

### ✅ 1. **Use Multiple Gates to Distribute Load**
```python
# Example:
# gate1 => 0–49 passwords
# gate2 => 50–99 passwords
```
- Assign a **unique gate** to each batch: `race1`, `race2`, `race3`, ...
- Call `engine.openGate("raceX")` after each batch is queued.

---

### ✅ 2. **Increase `concurrentConnections`**
```python
concurrentConnections = 5  # or higher based on testing
```
- Enables **parallel streams**.
- Helps avoid throttling and improves throughput.

---

### ✅ 3. **Avoid Assigning >100 Requests Per Gate**
- Try to keep batches around **90–100 requests max per gate** to prevent internal drop issues.
- Turbo Intruder performs better when batches are **smaller and structured**.

---

### ✅ 4. **Monitor Request Timing**
- If response timestamps appear **before** request timestamps:
  - 🕒 It indicates a **race failure** or **premature server response**.
  - Suggests the gate was not used correctly or the request was dropped.

---

### ✅ 5. **Consider Server-Side Throttling**
- Although less likely, some targets may:
  - 🛡️ **Rate-limit** connections
  - 🛡️ Block excessive request bursts
- Remediate by:
  - Adding small **delays**
  - Reducing **batch size**
  - Switching from `BURP2` to `THREADED` engine if needed.

---

> 💡 **Summary**: Always break requests into smaller groups with unique gates and ensure that you monitor Turbo Intruder's internal behavior. Tune the configuration based on how the target server handles stream multiplexing and rate limits.

---



# Conclusion:

It's crucial **`not to give up`** when faced with login rate limiting on applications. If default methods prove ineffective, there are **`numerous alternative ways`** to bypass these limitations. By thinking `outside the box` and creating new methods, it's possible to overcome even the most robust security measures. This adaptability underscores the importance of continuous **`vigilance and innovation`** in the field of security.

## References:

+ [Portswigger](https://www.linkedin.com/in/suresh-budarapu-74b5463b/)