---
title: "VLAN Hopping Attack"
tags:
    - VLAN
    - VLAN hopping
    - Network
    - Red Team
date: "2023-03-03"
thumbnail: "/assets/img/thumbnail/VLAN.gif"
---

# About the blog
---
* Author: [Nachiket Rathod](https://www.linkedin.com/in/nachiketrathod/)
* Genre: `Network Security`
* Objective: Delving into the VLAN hopping attack, a technique used to bypass network segmentation by sending packets to restricted ports.

<h3 id="tldr"> 
     <strong>GT;DR: </strong>
</h3>

This blog will `delve` into the `VLAN hopping` attack, which is a network attack technique that involves **sending packets** to a **port** that would typically be inaccessible from a specific end system. The attack targets virtual local area networks (VLANs), which are networks that group devices based on shared characteristics such as user type, department, or primary application rather than their physical location.

# Uncovering the Roots of VLAN Hopping Attacks

A VLAN hopping attack can `manifest` in two ways, one of which involves an attacker taking advantage of a **network switch** that's been set up for `autotrunking`. In this scenario, the attacker can trick the switch into appearing as if it has a **continuous need for trunking access** to all VLANs permitted on the trunk port.

# Dynamic Trunking Protocol [DTP]

Dynamic Trunking Protocol (DTP) is a trunking protocol developed by Cisco that enables **automatic negotiation** of `trunks` between Cisco switches. With DTP, switches can `dynamically negotiate` and `establish trunk connections` between them.

## Different Trunking Modes of DTP

| DTP Mode | Description |
| --- | --- |
| Dynamic Desirable (dynamic desirable) | In this mode, a switch actively tries to convert the link to a trunk link. It sends DTP frames advertising its capability to become a trunk and can also negotiate with the other end of the link. If the other end is set to "trunk" or "dynamic desirable", then a trunk link is formed. |
| Trunk (on) | In this mode, a switch will only create a trunk link with the other end of the link if the other end is set to "trunk" or "dynamic desirable". If the other end is set to "access", then a trunk link is not formed. |
| Dynamic Auto (dynamic auto) | In this mode, a switch passively waits for the other end of the link to initiate DTP negotiation. If the other end is set to "dynamic desirable" or "trunk", then a trunk link is formed. If the other end is set to "access" or "nonegotiate", then a trunk link is not formed. |
| Nonegotiate (nonegotiate) | In this mode, a switch disables DTP negotiation and will not form a trunk link with the other end of the link. |
| Access (access) | In this mode, a switch disables DTP negotiation and configures the link as an access port. The link cannot form a trunk link with the other end of the link. |

## Under What Conditions Can This Attack Be Performed?

For the attack to be successful, the **`switch mode`** must be configured as `dynamic desirable`, `dynamic auto`, or `trunk` to enable the switches to negotiate and exchange DTP packets. It's important to note that **Cisco switches** are typically set to `dynamic desirable` by default.

# Requirements for Demonstrating the Attack:

1. [GNS3](https://www.gns3.com/software/download)
2. [Kali Linux VM - [Attacker]](https://www.kali.org/get-kali/#kali-virtual-machines)
3. [Virtual Host - [Victim]](https://docs.gns3.com/docs/emulators/vpcs/)
4. [Switch - cisco-iosvl2 [Signin First]](https://gns3.com/marketplace/appliances/cisco-iosvl2)

#  **Starting the Process: 🪓**

* Consider a miniature network consisting of three clients, namely an `attacker` and **two** `victims`, all interconnected through a switch within the same network. To grasp the network's topology, please refer to the design diagram below:

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/1.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 1. All three clients two Victims and Attacker.</figcaption>
</figure>

+ The switch is connected to `PC-1` (**IP: 192.168.1.5**), `PC-2` (**IP: 10.0.0.4**), and the `Attacker's - KALI Machine`(**IP: 192.168.1.2**).

* The table presented below provides details about the clients and their respective VLAN IDs.

   | Device   | IP Address    | VLAN ID |
   |----------|---------------|---------|
   | PC-1     | 192.168.1.5   | 60      |
   | PC-2     | 10.0.0.4      | 90      |
   | Attacker | 192.168.1.2   | 60      | 

* Open the Console for all the three clients and switch - `cisco-iosvl2`

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/2.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 2. Console for PC-1, PC-2, PC-3 and the switch cisco-iosvl2.</figcaption>
</figure>

## Example:

  * Assume that the attacker has gained access to a network and is currently located in **`VLAN 60`**,  along with `PC-1` which is on the same subnet and VLAN. This implies that they are capable of pinging each other. However, `PC-2` [**`Victim`**] is located in a different subnet and has **`VLAN 90`**. Consequently, `PC-2` cannot ping either `PC-1` or the attacker. To test connectivity, we will perform a ping from both `PC-1` to the **attacker** (**`PC-3`**) and vice versa.<br>

1.) Open the console for PC-1, use the **`Show ip`** command and observe that **`no IP`** has been assigned to the `PC-1`. <br>
Now assign the IP with the below command:

  ```sass
   ip 192.168.1.2/24 gateway 192.168.1.1
  ```

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/3.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 3. Observe that IP address has been successfully assigned</figcaption>
</figure>

2.) Follow the same process as **`step-1`**, and assign the IP with the below command:

```sass
 ip 10.0.0.4/16 gateway 10.0.0.1
```

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/4.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 4. Observe that IP address has been successfully assigned</figcaption>
</figure>

3.) Repeat the same proceess as above for the attacker's **PC-3** [**`KALI`**], and ping from **PC-1** to **PC-3** and vice versa.

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/5.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 5. Observe the successfull ping</figcaption>
</figure>

4.) Repeat the `ping` process for the `attacker's` **`PC-3`** [**`KALI`**], to `Victim's` **`PC-2`** and vice versa.

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/6.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 6. As they're in different VLANs the ping will be unsuccessfull(obviously 😄)</figcaption>
</figure>

5.) Go to the **`Switch console`** and create **two VLANs** of different departments.
use below commands:

```bash
+ enable
+ configure terminal
+ vlan 60
+ name finance
+ exit
+ vlan 90
+ name security
+ exit/end
+ wr
```

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/7.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 7. VLAN 60 for finance and VLAN 90 for security</figcaption>
</figure>


6.) Set all the `gigabitEthernet interfaces` to **switchport mode access**.
use below command:

```sass
interface g0/0
switchport mode access
switchport access vlan 60
exit

interface g0/1
switchport mode access
switchport access vlan 60
exit

interface g0/2
switchport mode access
switchport access vlan 90
end

wr
```

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/8.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 8. Set switchport mode access for VLAN 60 and 90</figcaption>
</figure>

* The [switchport mode](https://www.connecteddots.online/resources/cisco-reference/switchport-mode-access#:~:text=The%20switchport%20mode%20command%20allows,for%20a%20single%20VLAN%20only.) command allows us to configure the trunking operational mode on a Layer 2 interface on a Cisco IOS device.

7.) Check the **status** for all the `VLANs` by using the below command:

```sass
show interface status
```

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/9.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 9. See that both the VLANs status are showing connected</figcaption>
</figure>

8.) Same as above check the **`gigabitEthernet`** trunk status. use the below command:

```sass
sh interfaces g0/0 trunk
sh interfaces g0/1 trunk
sh interfaces g0/2 trunk
```

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/10.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 10. Observe the different enties values e.g. Mode, Encapsulation, status.</figcaption>
</figure>

9.) In order to make the **attack successful**, the switch has to be on default configuration (in **`Dynamic Desirable`**), Indeed the switchport is set on `Dynamic Desirable` thus the VLANs can be `negotiated` together. let’s check the configuration of the attacker’s interface `(G0/0), (G0/1) and (G0/2)`:

```sass
sh interfaces switchport
```

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/11.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 11. Observe the Administrative Mode and Trunking Encapsulation on g0/0. </figcaption>
</figure>

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/12.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 12. Observe the Administrative Mode and Trunking Encapsulation on g0/1. </figcaption>
</figure>

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/13.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 13. Observe the Administrative Mode and Trunking Encapsulation on g0/2. </figcaption>
</figure>

10.) Now run the tool (**`yersinia`**) in order to enable the **TRUNK mode**, but before we run the attack let’s see the status of the VLAN:

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/14.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 14. Observe the status of VLANs. </figcaption>
</figure>

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/15.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 15. Run the yersinia, launch attack and select the enabling trunking. </figcaption>
</figure>

11.) As the VLANs are set correctly and we will run the **`debug mode`** to `see the incoming DTP packets`. Also see that there are packets have been sent as
shown below:

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/16.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 16. Run the yersinia, launch attack and select the enabling trunking. </figcaption>
</figure>

12.) Now check the VLAN table with below command:

```sass
sh interfaces g0/0 trunk
sh interfaces g0/1 trunk
sh interfaces g0/2 trunk
```
<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/17.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 17. Run the yersinia, launch attack and select the enabling trunking. </figcaption>
</figure>

```text
Note: We can see that the interface (G0/0) is set on trunk which means that we can jump other VLANs!
```

13.) Add the below commands to `KALI Machine`.

```sass
sudo modprob 8021q
sudo vconfig add eth0 90
```
<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/18.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 18. Add above commands in attacker's PC. </figcaption>
</figure>

14.) Now add a **new VLAN interface** and we gave it the `ID=90`. Then we added a new IP and make it up then assign the new created VLAN interface to the **`eth0.90`** interface and make up.

```sass
sudo ifconfig eth0.90 up
sudo ifconfig eth0.90 10.0.0.9 up
```

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/19.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 19. Assign the Id=90, IP=10.x.x.x and up the interface. </figcaption>
</figure>

15.) Finally, we can ping the **`PC-2`** that were not accessible and on other `VLAN(60)`.

<figure  style="text-align: center;">
<img src="/assets/blogs/VLAN/20.png" alt="picture" style="border:2px solid purple">
<figcaption style="color: red;">Fig 20. we successfully jumped to the VLAN (90)! </figcaption>
</figure>

# MITIGATION

VLAN Hopping can only be exploited when interfaces are set to negotiate a trunk. To prevent the VLAN hopping from being exploited, we can do the below mitigations:

+ Ensure that ports are not set to negotiate trunks automatically by disabling DTP:
```sass
Switch(config-if)# Switchport nonegotiate
```
* NEVER use **`VLAN 1`** at all.
* **Disable** `unused ports` and put them in an unused VLAN
* Always `use a dedicated VLAN ID` for all trunk ports.


## References:

- [AT&T](https://cybersecurity.att.com/blogs/security-essentials/vlan-hopping-and-mitigation)
- [NSS](https://notsosecure.com/exploiting-vlan-double-tagging)
- [YouTube](https://youtu.be/qaADvmUBbEA)
