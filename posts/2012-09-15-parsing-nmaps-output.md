---
layout: post
title: "Parsing Nmap's output"
date: 2012-09-15 18:10:26 -0400
comments: true
categories: boot2root
alias: /2012/09/parsing-nmaps-output.html
---

Nmap is a favorite tool when it comes to running port scans. The output can be a bit much however, especially when you're dealing with many targets with many services. Nmap is capable of producing reports in text, grepable, and XML formats. When I was working on my OSCP, I wanted a lightweight tool that could quickly parse my Nmap reports and display clean results. I couldn't find one that did what I wanted, so I hacked something together. The end result, is a script called scanreport.sh

<!--more-->

scanreport.sh reads an Nmap grepable output file and displays the results based on IP address, port number, or service. The tool is most helpful when you've got several machines that you've scanned, so as an example, we'll scan five machines and examine the output. The IP addresses of the targets are put in a targets.txt file: 

```
$ cat targets.txt 
192.168.81.171
192.168.81.182
192.168.81.143
192.168.81.119
192.168.81.190
```

We'll scan these IP addresses and save the output in grepable format, in a file called scan.txt: 

```
# nmap -sV -oG scan.txt -iL targets.txt 
 
Starting Nmap 6.01 ( http://nmap.org ) at 2012-09-15 10:17 EDT
Nmap scan report for 192.168.81.171
Host is up (0.049s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.3p1 Debian 3ubuntu4 (protocol 2.0)
80/tcp  open  http     Apache httpd 2.2.14 ((Ubuntu))
443/tcp open  ssl/http Apache httpd 2.2.14 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:kernel
 
Nmap scan report for 192.168.81.182
Host is up (0.040s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 5.3p1 Debian 3ubuntu4 (protocol 2.0)
80/tcp   open  http     Apache httpd 2.2.14 ((Ubuntu))
443/tcp  open  ssl/http Apache httpd 2.2.14 ((Ubuntu))
8080/tcp open  http     Apache Tomcat/Coyote JSP engine 1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:kernel
 
Nmap scan report for 192.168.81.143
Host is up (0.051s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0)
80/tcp  open  http     Apache httpd 2.2.16 ((Debian))
443/tcp open  ssl/http Apache httpd 2.2.16 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:kernel
 
Nmap scan report for 192.168.81.119
Host is up (0.055s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0)
80/tcp   open  http     Apache httpd 2.2.16 ((Debian))
443/tcp  open  ssl/http Apache httpd 2.2.16 ((Debian))
3690/tcp open  svnserve Subversion
8080/tcp open  http     Apache httpd 2.2.16 ((Debian))
9418/tcp open  git?
Service Info: OS: Linux; CPE: cpe:/o:linux:kernel
 
Nmap scan report for 192.168.81.190
Host is up (0.045s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0)
80/tcp   open  http     lighttpd 1.4.28
443/tcp  open  ssl/http lighttpd 1.4.28
3306/tcp open  mysql    MySQL 5.1.63-0+squeeze1
Service Info: OS: Linux; CPE: cpe:/o:linux:kernel
 
Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 5 IP addresses (5 hosts up) scanned in 21.29 seconds
```

If we look at the contents of the output file, scan.txt, we'll see that it contains Nmap's comments (lines starting with #): 

```
# Nmap 6.01 scan initiated Sat Sep 15 10:17:50 2012 as: nmap -sV -oG scan.txt -iL targets.txt
Host: 192.168.81.171 () Status: Up
Host: 192.168.81.171 () Ports: 22/open/tcp//ssh//OpenSSH 5.3p1 Debian 3ubuntu4 (protocol 2.0)/, 80/open/tcp//http//Apache httpd 2.2.14 ((Ubuntu))/, 443/open/tcp//ssl|http//Apache httpd 2.2.14 ((Ubuntu))/ Ignored State: closed (997)
Host: 192.168.81.182 () Status: Up
Host: 192.168.81.182 () Ports: 22/open/tcp//ssh//OpenSSH 5.3p1 Debian 3ubuntu4 (protocol 2.0)/, 80/open/tcp//http//Apache httpd 2.2.14 ((Ubuntu))/, 443/open/tcp//ssl|http//Apache httpd 2.2.14 ((Ubuntu))/, 8080/open/tcp//http//Apache Tomcat|Coyote JSP engine 1.1/ Ignored State: closed (996)
Host: 192.168.81.143 () Status: Up
Host: 192.168.81.143 () Ports: 22/open/tcp//ssh//OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0)/, 80/open/tcp//http//Apache httpd 2.2.16 ((Debian))/, 443/open/tcp//ssl|http//Apache httpd 2.2.16 ((Debian))/ Ignored State: closed (997)
Host: 192.168.81.119 () Status: Up
Host: 192.168.81.119 () Ports: 22/open/tcp//ssh//OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0)/, 80/open/tcp//http//Apache httpd 2.2.16 ((Debian))/, 443/open/tcp//ssl|http//Apache httpd 2.2.16 ((Debian))/, 3690/open/tcp//svnserve//Subversion/, 8080/open/tcp//http//Apache httpd 2.2.16 ((Debian))/, 9418/open/tcp//git?/// Ignored State: closed (994)
Host: 192.168.81.190 () Status: Up
Host: 192.168.81.190 () Ports: 22/open/tcp//ssh//OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0)/, 80/open/tcp//http//lighttpd 1.4.28/, 443/open/tcp//ssl|http//lighttpd 1.4.28/, 3306/open/tcp//mysql//MySQL 5.1.63-0+squeeze1/ Ignored State: closed (996)
# Nmap done at Sat Sep 15 10:18:11 2012 -- 5 IP addresses (5 hosts up) scanned in 21.29 seconds
```

These need to be removed before we can parse it with scanreport.sh. You can do this manually, or what I prefer, is to use grep and redirect the non-commented lines to another file, such as: 

```
grep -v ^# scan.txt > report.txt
```

Once the report is ready, we can use scanreport.sh to display the results: 

```
# scanreport.sh -f report.txt 
 
Host: 192.168.81.171 () 
22 open tcp  ssh  OpenSSH 5.3p1 Debian 3ubuntu4 (protocol 2.0) 
80 open tcp  http  Apache httpd 2.2.14 ((Ubuntu)) 
 
Host: 192.168.81.182 () 
22 open tcp  ssh  OpenSSH 5.3p1 Debian 3ubuntu4 (protocol 2.0) 
80 open tcp  http  Apache httpd 2.2.14 ((Ubuntu)) 
443 open tcp  ssl|http  Apache httpd 2.2.14 ((Ubuntu)) 
 
Host: 192.168.81.143 () 
22 open tcp  ssh  OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0) 
80 open tcp  http  Apache httpd 2.2.16 ((Debian)) 
 
Host: 192.168.81.119 () 
22 open tcp  ssh  OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0) 
80 open tcp  http  Apache httpd 2.2.16 ((Debian)) 
443 open tcp  ssl|http  Apache httpd 2.2.16 ((Debian)) 
3690 open tcp  svnserve  Subversion 
8080 open tcp  http  Apache httpd 2.2.16 ((Debian)) 
 
Host: 192.168.81.190 () 
22 open tcp  ssh  OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0) 
80 open tcp  http  lighttpd 1.4.28 
443 open tcp  ssl|http  lighttpd 1.4.28 
```

Without specifying anything other than the report file, we get a listing of every target and open ports that were discovered. We can be more specific and ask for only open ports from 192.168.81.119: 

```
# scanreport.sh -f report.txt -i 192.168.81.119
 
Host: 192.168.81.119 () 
22 open tcp  ssh  OpenSSH 5.5p1 Debian 6+squeeze2 (protocol 2.0) 
80 open tcp  http  Apache httpd 2.2.16 ((Debian)) 
443 open tcp  ssl|http  Apache httpd 2.2.16 ((Debian)) 
3690 open tcp  svnserve  Subversion 
8080 open tcp  http  Apache httpd 2.2.16 ((Debian)) 
```

Let's say we want to know which targets have port 3690 open: 

```
# scanreport.sh -f report.txt -p 3690
 
Host: 192.168.81.119 () 
3690 open tcp  svnserve  Subversion 
```

We can also look for a specific service by name. Let's say we want to find hosts running lighttpd: 

```
# scanreport.sh -f report.txt -s lighttpd
 
Host: 192.168.81.190 () 
80 open tcp  http  lighttpd 1.4.28 
443 open tcp  ssl|http  lighttpd 1.4.28 
```

It's easy enough to modify the script and add additional functionality. Feel free to modify it and adapt it to your needs. You can grab the latest version from [GitHub](https://github.com/superkojiman/scanreport)

I'm sure the script is not without some bugs, however for the most part I've had no issues with it, and I've found it helpful when combined with [onetwopunch.sh](/2012/05/31/port-scanning-one-two-punch/).
