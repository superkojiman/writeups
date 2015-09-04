---
layout: post
title: "De-ICE hacking challenge: Part 4"
date: 2013-12-13 18:10:26 -0400
comments: true
categories: boot2root
alias: /2013/12/de-ice-hacking-challenge-part-4.html
---

This is a quick walkthrough on solving the De-ICE S1.120 A challenge which can be downloaded here: [http://vulnhub.com/entry/de-ice_s1120-a,10/](http://vulnhub.com/entry/de-ice_s1120-a,10/). Interestingly, I wasn't aware that this boot2root even existed until a couple of nights ago when someone mentioned it on IRC. After doing a bit of searching, it turns out there are at least three that I haven't had a go at popping. So with that in mind, I decided to load up S1.120 A and take the challenge. 

<!--more-->

De-ICE S1.120 A has a static IP address of 192.168.1.120, so configure your network accordingly to acommodate it. I ran a thorough port scan on the server to identify open TCP and UDP ports: 

->![](/images/2013-12-13/01.png)<-

After a few minutes, TCP ports 21, 22, and 80 were identified, and no UDP ports were found to be open. I fired up my web browser and navigated to http://192.168.1.120 to have a look at the website.

->![](/images/2013-12-13/02.png)<-

The hyperlinks at the bottom of the wepbage allowed the user to add items into the database and to view items that were in the database. I examined each of these starting with the Add Product link. 

->![](/images/2013-12-13/03.png)<-

I added some random product to see what would happen. I submitted the form and got confirmation that it was added into the database. Since I added text to the price field instead of a numerical price, it just set it to $0.00. 

->![](/images/2013-12-13/04.png)<-

Moving on to the View Products page, I noticed that the dropdown menu had the item that I just added. 

->![](/images/2013-12-13/05.png)<-

Clicking submit returned the information for that item, which was basically what I had entered earlier. 

->![](/images/2013-12-13/06.png)<-

One interesting thing I noticed was the URL had changed to the following:

```
http://192.168.1.120/products.php?id=1
```

I had a hunch there might be some SQL injection vulnerability on the target, so I started enumerating it manually at first, but got no results. Thinking it might be vulnerable to blind SQL injection, I fired up sqlmap to do the heavy lifting for me. 

I ran a simple scan just to see if this worked:

->![](/images/2013-12-13/07.png)<-

Sure enough, the id parameter was vulnerable to SQL injection, and sqlmap identified the database as MySQL 5.1.33. 

->![](/images/2013-12-13/08.png)<-

I followed up by dumping usernames and hashes:

->![](/images/2013-12-13/09.png)<-

A huge number of users were identified: 

```
[11:44:36] [INFO] the back-end DBMS is MySQL
web application technology: Apache 2.2.11, PHP 5.2.9
back-end DBMS: MySQL 5.0.11
[11:44:36] [INFO] fetching database users
database management system users [50]:
[*] 'aadams'@'localhost'
[*] 'aallen'@'localhost'
[*] 'aard'@'localhost'
[*] 'aharp'@'localhost'
[*] 'aheflin'@'localhost'
[*] 'amaynard'@'localhost'
[*] 'aspears'@'localhost'
[*] 'aweiland'@'localhost'
[*] 'bbanter'@'localhost'
[*] 'bphillips'@'localhost'
[*] 'bwatkins'@'localhost'
[*] 'cchisholm'@'localhost'
[*] 'ccoffee'@'localhost'
[*] 'dcooper'@'localhost'
[*] 'dgilfillan'@'localhost'
[*] 'dgrant'@'localhost'
[*] 'djohnson'@'localhost'
[*] 'dstevens'@'localhost'
[*] 'dtraylor'@'localhost'
[*] 'dwestling'@'localhost'
[*] 'hlovell'@'localhost'
[*] 'jalcantar'@'localhost'
[*] 'jalvarez'@'localhost'
[*] 'jayala'@'localhost'
[*] 'jbresnahan'@'localhost'
[*] 'jdavenport'@'localhost'
[*] 'jduff'@'localhost'
[*] 'jfranklin'@'localhost'
[*] 'kclemons'@'localhost'
[*] 'krenfro'@'localhost'
[*] 'ktso'@'localhost'
[*] 'kwebber'@'localhost'
[*] 'lmartinez'@'localhost'
[*] 'lmorales'@'localhost'
[*] 'mbryan'@'localhost'
[*] 'mholland'@'localhost'
[*] 'mnader'@'localhost'
[*] 'mrodriguez'@'localhost'
[*] 'myajima'@'localhost'
[*] 'qpowers'@'localhost'
[*] 'rdominguez'@'localhost'
[*] 'rjacobson'@'localhost'
[*] 'rpatel'@'localhost'
[*] 'sgains'@'localhost'
[*] 'sjohnson'@'localhost'
[*] 'strammel'@'localhost'
[*] 'swarren'@'localhost'
[*] 'tdeleon'@'localhost'
[*] 'tgoodchap'@'localhost'
[*] 'webapp'@'localhost'
```

sqlmap asked if I wanted to save the hashes for later cracking, to which I did. I wanted to crack these myself with john 

->![](/images/2013-12-13/10.png)<-

The hashes were saved to /tmp/sqlmaphashes-OIvzPL.txt. I fired up john against the hash file and within a few seconds, 46 of the 50 hashes had already been cracked:

->![](/images/2013-12-13/11.png)<-

If all of these users had the same password for their SSH login, and if they had user accounts on the server, then I should be able to login now. I picked the first user on the list of cracked accounts, dwestling with password 123456 and logged in usng SSH:

->![](/images/2013-12-13/12.png)<-

This user had no sudo privileges, but with a shell on the target, I was able to do some more enumeration. Looking at the contents of /etc/group, I noticed that user ccoffee was part of the admin group. 

->![](/images/2013-12-13/13.png)<-

I checked the cracked passwords, and found that ccoffee's password was master. I logged out the current SSH session and logged back in as ccoffee. Checking sudo -l revealed that ccoffee could run a script getlogs.sh as root:

->![](/images/2013-12-13/14.png)<-

Running getlogs.sh returns the word "wrong". Unfortunately the permissions on the script prevented me from checking to see what it actually did. It occured to me that I didn't actually need to know what this script did, I could simply replace it with something else. I had write access to the /home/ccoffee/scripts directory, so I simply renamed it and created my own scripts directory with a getlogs.sh symlink to /bin/sh

->![](/images/2013-12-13/15.png)<-

Having symlinked /bin/sh to getlogs.sh, I executed sudo scripts/getlogs.sh and obtained my root shell:

->![](/images/2013-12-13/16.png)<-

I wasn't sure what the end goal of this challenge was. According to the flags section in [VulnHub](http://vulnhub.com/entry/de-ice_s1120-a,10/), it says various "internal documents". As root I found a personnel.doc in /home/ktso, and a bunch of other text files scattered around other user's home directories. 

Overall this was an easy challenge, which makes it well suited for beginners. I would say having a good knowledge of Linux and the basics of hacking techniques and tools would be enough to complete it. 
