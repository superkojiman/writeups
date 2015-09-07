---
layout: post
title: "Weaponizing the Blackberry"
date: 2009-10-09 18:14:37 -0400
comments: true
categories: hacking coding
alias: /2009/10/weaponizing-blackberry-part-1.html
---

I recently purchased a Blackberry on eBay. I needed a new phone and I
thought a smartphone might be nice to play with. The Blackberry Pearl 8120
has built in wifi capabilities which means you can bum off a wireless
network at Starbucks without paying for a data plan. This also means that
if we join the wireless network, we can "see" what other computers have
joined in as well and even probe them for interesting information. 

<!--more-->

I decided to write a basic port scanner. I've never done J2ME programming
before so I thought this would be great way to get my feet wet. I admit
that due to my excitement I flew through most of the Blackberry and J2ME
documentation and wrote the port scanner out of bits and pieces of the API.
The end result?

It works. 

I call it Dingleberry (name subject to change in the future... maybe). It
needs work and cleanup but as proof of concept it passes. 

When the application is started, it displays the IP address assigned by the
DHCP server. Armed with this information you can now enter in the IP
address of the host and the range of ports to scan. 

The algorithm I'm using is simple. Each port to be scanned is stored in an
array. A thread is started for each port which performs the TCP connection
to that port. By giving each connection its own thread the scan goes much
faster since we don't have to wait for the results of the current
connection to finish. Just start it, move on, and check the results when
we're done processing all the ports in the array. 

There are limitations. It's not as fast as I'd like it to be, scanning only
10 ports per second. For a small range of ports that's fine, but for a
larger range, it could take a while. A full TCP connect also means it's
noisy on the target's logs and if there's an IDS/IPS installed on the
target, it'll trigger it for sure.

Future plans:

* Now that the excitement of seeing whether or not this was possible in the
first place has worn off, I can work on improving the code, making it
faster and more efficient.

* Currently it just displays the port number. That's great if you know what
port 31765 is. I don't. I'm thinking of including a services file that the
application can reference to determine the actual service, or even doing a
banner grab on the port.

* SYN/ACK scans. Is it even possible to craft my own packets with J2ME?

Lots more work to go into it, and eventually I'll post the code once it's
stable enough. I think it makes a pretty neat reconnaissance device. Port
scanning with a tiny device in your pocket is a lot more discreet than
doing it with a laptop.
