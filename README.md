# Hacking Network Protocols: DHCP

In this comprehensive project i will cover what is DHCP and how to perform some basic DHCP attack’s. Through this project i will be using:

- **GNS 3**
- **PC1 with Ubuntu**
- **Rogue PC with Kali Linux**
- **Cisco Switch and Router**
- **Wireshark /tcpdump for packet analysis**

# Part 1 - DHCP

*Imagine this:* You're stepping into your favorite coffee shop or visiting a friend. The aroma of coffee fills the air or the laughter of good company surrounds you. You pull out your phone, eager to connect to the available Wi-Fi. A quick password entry, and voilà, you're online! Browsing, streaming, chatting - the digital world at your fingertips.

But have you ever wondered how this seamless connection happens? What magic lets you dive into the internet ocean without a hitch?

Well, it's not just magic; it's technology. And a key player in this process is **DHCP** - *Dynamic Host Configuration Protocol*. In this project i will demonstrate how DHCP works and how crucial is in modern network topologies.

DHCP is a network protocol which is responsible for allocating IP addresses in the network. Due this reason when u connect to a Wi-Fi u are able to browse out of the box - your device has allocated IP address in the network, your device is able to know where to send traffic when want to browse outside of your local network (for example Facebook) and also knows which DNS to use ( DNS is technology which allow us to use human readable names not IP addresses ).

DHCP is built on a client-server model, where designated DHCP server hosts allocate network addresses and deliver configuration parameters to dynamically configured hosts.

## Simple DHCP configuration

Lets configure basic network with DHCP, in this case our DHCP server will be our gateway (Router).

We have simple network topology - PC1, Switch and Router and connection to the Internet.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled.png)

## Configure DHCP

First of all we need to exclude two IP addresses from the topology - IP for Router and for the Switch. Basic switch usually do not have IP address because is operating on Data-link layer of OSI model (L2) and practically do not need one in this simple topology but we will assign it anyway. 

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%201.png)

Cool, lets configure the pool, add default router and also DNS. I will show really quickly why DNS is important to specify in the configuration, lets save the config and test our work.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%202.png)

I will drop the DHCP lease on the PC1 using dhclient command, and request a new IP address with same command from the DHCP server. Ill then run the Wireshark to show you the process under the hood.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%203.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%204.png)

Based on the capture we can see PC1 is using 0.0.0.0 address to send DHCP discover message. DHCP is located in Layer 7 (application) but PC1 has no IP address to participate in the IP stack, so DHCP is using temporary 0.0.0.0 to send broadcast (to all members in the network) to discover the DHCP server. 

If we have DHCP server configured it will reply with DHCP Offer.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%205.png)

Once client accept the offer, we can send the DHCP request to the server to allocate offered address.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%206.png)

Once DHCP server will acknowledge our request, it will send ACK packet to the client like this.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%207.png)

DHCP process is done, PC1 has:

- IP address
- default route configured so we can ping Google DNS which means we can reach out network outside of our subnet
- can browse the web (hence DNS is configured)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%208.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%209.png)

## DHCP options

Notice the Option tabs on the bottom of the DHCP packet. This is additional options which DHCP server may have configured. For example we can see the Option(6) DNS or Option (51) IP Address Lease Time. Lets break down this two.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2010.png)

### Lease time

In today’s world we can lease almost everything - car, any electronic device such iPhone and also 

IP address. Lease time shows how long we can have our IP address allocated. The purpose of the lease time is to efficiently manage a limited pool of IP addresses in a dynamic network environment where devices frequently connect and disconnect. It ensures that IP addresses are not permanently tied up with devices that no longer need them or are no longer present on the network. Lease time in our case is set for one day - default for Cisco devices. 

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2011.png)

Lets extend the DHCP lease time to 5 days and renew the PC1 DHCP configuration again and capture the traffic.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2012.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2013.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2014.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2015.png)

As we can see our new lease time is 5 days, success.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2016.png)

### DNS

DNS is topic for another project but without DNS the todays Internet cannot exist. DNS in very simple words translate human readable addresses such as [google.com](http://google.com) to IP address and vice versa. Why does exist? Simple - TCP/IP does not understand human readable text - it does understand only IP addresses. (no shit Sherlock since is TCP/IP). So another option we can offer during IP address allocation from DHCP server is DNS server. 

Since we have DNS server setup already on DHCP server we will turn it off and renew the address on PC1 and see that happens. 

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2017.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2018.png)

Success, we have another IP adress and we can ping the gateway and also google DNS.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2019.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2020.png)

In the Wireshark capture we can see full DHCP process, but there is absence of DNS option.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2021.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2022.png)

So if we want to browse the Internet via hostname on PC1 no luck. That is why is DNS so important during DHCP offer, so lets restore the original configuration.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2023.png)

This was brief introduction to the DHCP and hence end of the Part 1, in Part we will focus on some of the common DHCP attacks.

# Part 2 - Hacking DHCP

**!!! DISCLAIMER - this section is dedicated to some of the common DHCP attacks. Performing these attacks are only for educational purposes. I am performing these attacks in the isolated lab which i own and manage. DO NOT PERFORM this type of attacks on any network which you do not own otherwise this will be considered illegal, un-ethical and may result in pressing charges and coliding with authorities in particullar organization or state.**

## Gear up

Ok we wrapped up the theory of DHCP and how is operating. Lets have some fun now. First of all we need to setup Kali Linux - Kali is number one Linux distribution when it comes to hacking or pentesting - you have all tools already pre-installed and thoose we dont have can be easily installed from Kali repo’s. 
Ill suggest to follow official documentation how to setup Kali inside GNS3 here:
[https://www.gns3.com/marketplace/appliances/kali-linux-2](https://www.gns3.com/marketplace/appliances/kali-linux-2)

Lets connect and power up our Kali or better named Rogue to our topology. To any device in the topology it appears as standard client PC - it has ip adress, can browse the web, reach out to any device inside the network.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2024.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2025.png)

We are ready to go - one of the framework’s used to hack the network protocols is **Yersinia**.

**Yersinia** covers much more than DHCP - hacking common Cisco protocols such as CDP, HSRP or STP, DTP, VTP are part of the Yersinia’s framework as well. For more details check the documentation here:

[https://www.kali.org/tools/yersinia/](https://www.kali.org/tools/yersinia/)

[https://github.com/tomac/yersinia](https://github.com/tomac/yersinia)

## DHCP DDoS

Simple DDoS (Denial of Service) attack can be executed using Yersinia. This attach will send hundreds of DHCP discover messages which will totally exhaust the pool of IP addresses and will deny any DHCP renewal. Lets give it a try:

Start Wireshark capture on Rogue interface - for now there is no traffic - since DHCP process was sucessfull upon boot.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2026.png)

Lets run Yersinia in Interactive mode and pick DHCP and press 1 for sending DISCOVER packets.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2027.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2028.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2029.png)

Once we launch the attack we can already see the hundreds of DHCP packets

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2030.png)

Now lets release DHCP on the PC1 and try to renew the adress allocation.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2031.png)

I guess no luck - and this is why. DHCP DDoS totally flooded DHCP server with Discover messages resulting in total pool exhaustion.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2032.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2033.png)

This DDoS is also an Broadcast Storm - since DHCP discover is Broadcast - sending packets to every device in the network - it will also result in higher network latency and network slowness. Lets test ping from Switch to Google DNS.

We can spot the intermittent timeouts inside the icmp messages.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2034.png)

Also Broadcast has another dark side - every network device HAS to process broadcast. And this is by design - protocols which are using broadcasts (such as DHCP, ARP) relying on the processed messages by network devices. This creates attack vector.

Router’s CPU need to process every broadcast sent to the network. This is also adding more latency and can result in very high CPU utilization on network devices hence causing overheat and physical damage to the devices.

CPU utilization on Router - we can see nearly 25% of Router CPU is occupied by DHCP requests and over 50% is occupied by IP Input - in our case Broadcast messages.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2035.png)

Ok that was an simple DHCP DDoS which can do pretty decent harm. Lets stop the attack and verify we are operational as before.

Router CPU is almost 0% now and PC1 has IP address and IP connectivity is functional

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2036.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2037.png)

## Rogue DHCP

Another type of attack is called Rogue DHCP - this is type of MITM (Man in the Middle) attack. This attack will create fake (Rogue) DHCP server which will respond to DHCP queries from clients + will forward all traffic via itself. This means any connection from clients > Internet will go via our Rogue DHCP server and Rogue will forward it further to his default-gateway - in our case Router.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2038.png)

This also means we can sniff or tap to any communication which will PC1 establish - this is the main purpose of this attack - to sit in the middle and steal any valuable information. This is why you should never trust to any Public Free Wi-fi  or browse/establish any un-encrypted connection such as http or telnet and will soon show you why. 

During this attack we will use Ettercap - easy to use tool for MiTM attack such Rogue DHCP. 

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2039.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2040.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2041.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2042.png)

Ok attack is running, lets request a new address from DHCP server and also open Wireshark in GNS3. 

We can see we still have address from Router - lets request a new from DHCP server. 

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2043.png)

As we can see the new address is allocated but from the Rogue DHCP pool starting from 10.0.0.100 - we can see the default-route also changed. Wireshark capture and Ettercap confirms this also:

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2044.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2045.png)

Now when we want to access the Internet connection is flowing from PC1 > Rogue > Router > Internet - simple ping and tcpdump will show us the flow. tcpdump is simmilar to Wireshark but is cLI only - very powerful tool for analyzing traffic. 

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2046.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2047.png)

Ok we are all set - lets have some fun.

### Telnet

Telnet in these days is used mainly for port testing - we can simply test if the connection is open on the other side in case of any connection issues - very useful and easy. But telnet in the old days was the standard for remote connection and management via cli (command line interface). Telnet is easy to use and establish but it has a one big flaw - communication is in plain text. This means anyone who sits in the middle will be able to see everything. 

Lets imagine we are using telnet in our network to manage Router via cLI. We will enable telnet on device and also spin up Loopback interface - Loopback’s are mainly used for remote management or Cisco routers but this is topic for another project.

We have Loopback interface up and running, lets test connection from PC1 > Loopback.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2048.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2049.png)

Cool, connection is working, lets telnet to Router and meanwhile run Wireshark on Rogue server.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2050.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2051.png)

We can immediately see the telnet packets on Rogue server.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2052.png)

And when we follow TCP stream - basically this will reconstruct the stream packet by packet, since every character is single packet this will construct the whole stream so we can see it - we will see whole conversation with password.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2053.png)

Upon checking the Router telnet configuration password is matched with the one we captured

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2054.png)

Now you can see how easy is to capture telnet traffic and re-use it. Do not use telnet.

### HTTP

HTTP protocol is the backbone of the modern web. Every page and every API request is using http. But unlike https which came after http, http has same flaw as telnet - its using plaintext. Lets enable simple http server on the Router, use browser on the PC1 to access the web-page and capture traffic with Rogue.

Enabling http server and setting up the authentication method as local - this means authentication will go via Router account which we will create. As we can see we created account admin with full privileges and password - password is hashed in this case so anyone who standing behind us will not see the password so there is partial security in this case.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2055.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2056.png)

Lets test open port with telnet from PC1- standard port for HTTP is 80.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2057.png)

Good - now we will fire up Wireshark on Rogue PC and filter for http traffic. On PC1 we will connect to http server via browser.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2058.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2059.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2060.png)

We successfully captured the HTTP traffic and connected to the Router web interface. Lets open up the http packet and see if we captured some password.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2061.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2062.png)

Pretty scary - we can easily capture credentials and re-use it to do decent harm. As you saw this account has full privileges which means we can easily reconfigure the Router or completely shutdown the network per our will. 

Now I will combine credentials from telnet hack and http hack to gain the admin access to the Router.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2063.png)

We are in, now lets cut off the Internet access, as we can see Router is using G0/1 Interface to access the Internet.

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2064.png)

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2065.png)

Now the Google DNS will not respond, we easily gained access to the Router and disrupted the network. 

![Untitled](Hacking%20Network%20Protocols%20DHCP%202421f485f1324769bd6d9182748a6425/Untitled%2066.png)

### Wrap up

With few hacks we were able to cause decent harm in the network. So things to understand are:

- rather than telnet use ssh - rather disable telnet completely
- rather than http use https - in these days https is a mandatory for most sites hosted on the Internet but there are attacks such as Downgrade attack which will force site to use http over https - in this case when such attack is out there use VPN - VPN will encrypt the connection from endpoint to endpoint which means from our PC to the target server whole connection is encrypted
- use strong password for devices in the network topology and do not reuse password for multiple accounts

And that’s it for DHCP Hacking, if you made it till this point thank you for your audience, i hope you found this interesting and educational in some way. Cheers 👌