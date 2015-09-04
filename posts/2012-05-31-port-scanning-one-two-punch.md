---
layout: post
title: "Port scanning one, two punch"
date: 2012-05-31 18:10:26 -0400
comments: true
categories: hacking coding
alias: /2012/05/port-scanning-one-two-punch.html
---

Information gathering is an important step in a penetration test, or any hack attempt. Various attack vectors open up based on the findings in the information gathering stage. Port scanning provides a large amount of information on open services and possible exploits that target these services. The problem with port scanning is that it can take a lot of time to generate the results depending on the type of scan, the protocol that's being scanned, the number of targets, whether or not any IDS is in the way, and a slew of other variables.

<!--more-->

Nmap is by far the most comprehensive port scanner, able to identify services, fingerprint operating systems, and even run several scripts against the services to identify potential vulnerabilities. This helps cut down the manual work involved in service enumeration. Nmap uses a default list of ports when none are provided by the attacker. This can cause nmap to miss certain ports that are not in its default list. There is the option to let nmap scan all 65,535 ports on each machine, but as you can imagine, this will take a considerable amount of time, especially if you're scanning a lot of targets.

Unicornscan is another port scanner that utilizes it's own userland TCP/IP stack, which allows it to run a asynchronous scans. This makes it a whole lot faster than nmap and can scan 65,535 ports in a relatively shorter time frame. Since unicornscan is so fast, it makes sense to use it for scanning large networks or a large number of ports.

So, the idea behind this post is to utilize both tools to generate a scan of 65,535 ports on the targets. We will use unicornscan to scan all ports, and make a list of those ports that are open. We will then take the open ports and pass them to nmap for service detection. You can grab the latest version from [GitHub](https://github.com/superkojiman/onetwopunch). 

Let's see this in action. We create a text file, lab.txt, containing a list of our targets:

```
192.168.1.144
192.168.1.179
192.168.1.182
```

We'll run onetwopunch.sh and tell it to read lab.txt and perform a TCP scan against each target.

```
# ./onetwopunch.sh lab.txt tcp
[+] scanning 192.168.1.144 for tcp ports...
[+] obtaining all open tcp ports using unicornscan...
[+] unicornscan -msf 192.168.1.144:a -l udir/192.168.1.144-tcp.txt
[+] ports for nmap to scan: 22,80,111,139,443,1024,
[+] nmap -sV -oX ndir/192.168.1.144-tcp.xml -oG ndir/192.168.1.144-tcp.grep -p 22,80,111,139,443,1024, 192.168.1.144
 
Starting Nmap 5.61TEST4 ( http://nmap.org ) at 2012-05-31 13:54 EDT
Nmap scan report for 192.168.1.144
Host is up (0.033s latency).
PORT     STATE SERVICE              VERSION
22/tcp   open  ssh                  OpenSSH 2.9p2 (protocol 1.99)
80/tcp   open  http                 Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp  open  rpcbind (rpcbind V2) 2 (rpc #100000)
139/tcp  open  netbios-ssn          Samba smbd (workgroup: MYGROUP)
443/tcp  open  ssl/http             Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
1024/tcp open  status (status V1)   1 (rpc #100024)
MAC Address: 00:0C:29:46:F7:8C (VMware)
 
Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.62 seconds
 
[+] scanning 192.168.1.179 for tcp ports...
[+] obtaining all open tcp ports using unicornscan...
[+] unicornscan -msf 192.168.1.179:a -l udir/192.168.1.179-tcp.txt
[+] ports for nmap to scan: 22,3389,10000,
[+] nmap -sV -oX ndir/192.168.1.179-tcp.xml -oG ndir/192.168.1.179-tcp.grep -p 22,3389,10000, 192.168.1.179
 
Starting Nmap 5.61TEST4 ( http://nmap.org ) at 2012-05-31 13:58 EDT
Nmap scan report for 192.168.1.179
Host is up (0.00041s latency).
PORT      STATE    SERVICE          VERSION
22/tcp    open     ssh              WeOnlyDo sshd 1.2.7 (protocol 2.0)
3389/tcp  open     microsoft-rdp    Microsoft Terminal Service
10000/tcp filtered snet-sensor-mgmt
MAC Address: 00:0C:29:F2:D6:40 (VMware)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
 
Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.53 seconds
 
[+] scanning 192.168.1.182 for tcp ports...
[+] obtaining all open tcp ports using unicornscan...
[+] unicornscan -msf 192.168.1.182:a -l udir/192.168.1.182-tcp.txt
[+] ports for nmap to scan: 22,111,446,969,3260,
[+] nmap -sV -oX ndir/192.168.1.182-tcp.xml -oG ndir/192.168.1.182-tcp.grep -p 22,111,446,969,3260, 192.168.1.182
 
Starting Nmap 5.61TEST4 ( http://nmap.org ) at 2012-05-31 14:02 EDT
Nmap scan report for 192.168.1.182
Host is up (0.033s latency).
PORT     STATE SERVICE              VERSION
22/tcp   open  ssh                  OpenSSH 4.9 (protocol 2.0)
111/tcp  open  rpcbind (rpcbind V2) 2 (rpc #100000)
446/tcp  open  http                 Apache httpd (SSL-only mode)
969/tcp  open  status (status V1)   1 (rpc #100024)
3260/tcp open  iscsi?
MAC Address: 00:0C:29:C9:7C:C9 (VMware)
 
Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 89.11 seconds
 
[+] scans completed
```

There we have it. Using unicornscan we obtain a list of all open TCP ports, and then pass that list to nmap for a service scan. We can modify the script further to enable more aggressive scanning on nmap's part, and even run several NSE scripts. Keep in mind though that the more power you give nmap, the longer it can take to complete the scans. It might be better to just use this as a quick enumeration tool, and then run aggressive scans on interesting looking ports. 
