---
layout: post
title: "Brainpan hacking challenge"
date: 2013-03-20 18:10:26 -0400
comments: true
categories: boot2root
alias: /2013/03/brainpan-hacking-challenge.html
---

After attempting various hacking challenges, I was inspired to come up with my own. Brainpan is my attempt at a vulnerable virtual machine. Your goal is to break in and get root access. 

<!--more-->

>By using this virtual machine, you agree that in no event will I be liable for any loss or damage including without limitation, indirect or consequential loss or damage, or any loss or damage whatsoever arising from loss of data or profits arising out of or in connection with the use of this software.

TL;DR: If something bad happens, it's not my fault. 
Brainpan has been tested and found to work on the following hypervisors:

* VMware Player 5.0.1
* VMWare Fusion 5.0
* VirtualBox 4.2.8

Import brainpan.ova into your preferred hypervisor and configure the network settings to your needs. It will get an IP address via DHCP, but it's recommended you run it within a NAT or visible to the host OS only since it is vulnerable to attacks.

[VulnHub](http://vulnhub.com/) has been kind enough to host the virtual machine, so many thanks to them! Download it here: [http://vulnhub.com/entry/brainpan_1,51/](http://vulnhub.com/entry/brainpan_1,51/) 

It's also mirrored at [Google Drive](https://drive.google.com/file/d/0B41M3Dojh4xbVE9zeU5DNTRCT1E/edit?usp=sharing), although be warned that Google imposes bandwidth limits.
