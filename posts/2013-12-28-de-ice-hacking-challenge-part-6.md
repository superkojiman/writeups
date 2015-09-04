---
layout: post
title: "De-ICE hacking challenge: Part 6"
date: 2013-12-28 02:35:09 -0400
comments: true
categories: boot2root
alias: /2013/12/de-ice-hacking-challenge-part-6.html
---

This is a walkthrough on De-ICE S1.140, available for download at [VulnHub](http://vulnhub.com/entry/de-ice_s1140,57/). This release was much anticipated and took a while to get released to the public. It's a little tougher than the previous De-ICE challenges, but uses a similar formula of password cracking and guessing. 

<!--more-->

After booting up the De-ICE virtual machine, I used netdiscover and identified the target's IP address as 192.168.1.146. I copied the IP address into a file, t.txt, and passed it into onetwopunch.sh for a full TCP and UDP port scan:

->![](/images/2013-12-28/01.png)<-

Several services were found running: FTP, SSH, HTTP, IMAP, and POP3. I pointed my web browser to http://192.168.1.146 and was greeted by a the Lazy Admin Corp. webpage:

->![](/images/2013-12-28/02.png)<-

Aside from some hints for completing the challenge, there wasn't much else on the webpage. At this point I fired up nikto and wfuzz to enumerate the website further. 

->![](/images/2013-12-28/03.png)<-

->![](/images/2013-12-28/04.png)<-

Both programs identified a new resource, /forum. Navigating to it, I found a web forum powered by my little forum. I did a quick search online for any vulnerabilities affecting this version of my little forum but found nothing of use. 

->![](/images/2013-12-28/05.png)<-

Looking through the forum threads, I discovered that one of the users, Sandy Willard used to have a different surname, Raines. I also found a thread containing a snapshot of what appeared to be an SSH log on the server. There were several user names on the list, and I thought it might be a good idea to just pull them out and pop them into a text file in case I needed to use it for hydra. I wrote a quick shell script to do this, but the results produced a little gem:

->![](/images/2013-12-28/06.png)<-

One of the user names recorded in the log was !DFiuoTkbxtdk0!. This appeared to be a password. Most likely the user attempted to SSH in, but typed his password first instead of his user name. Looking at the log again, I noticed that there was only one successful login, which was from user mbrown

->![](/images/2013-12-28/07.png)<-

Naturally the first thing I did was to attempt to SSH to the server. This failed miserably, with the SSH server disallowing password authentication. I needed an SSH key to login to the server. I compiled a list of users based on the names I gathered on the forum and tried to see if I had access to their home directories, eg: http://192.168.1.146/~mbrown/.ssh. This exercises proved to be in vain. 

I attempted to login to the forum as mbrown with the password I found, and it worked. I explored the forum a little bit more as an authenticated user, and found mbrown's email address on his profile page:

->![](/images/2013-12-28/08.png)<-

I didn't see anything else of interest in the forum, so I set my eyes on the next service: FTP. Attempting to login as mbrown with his password didn't work. However, anonymous login was allowed. I found a directory called incoming which didn't appear to contain anything. 

->![](/images/2013-12-28/09.png)<-

Attempting to upload anything in incoming didn't work. Perhaps this user didn't have the proper permissions to write to that directory. The next two services were IMAP and POP3, both of which were running with SSL. I decided to try IMAP first and connected to the service using the following command: 

```
# openssl s_client -connect 192.168.1.146:993 -crlf
```

In order to proceed, I needed to authenticate. I was able to login using the email address for mbrown listed on his forum profile, and his forum password:

->![](/images/2013-12-28/10.png)<-

Having logged in, I listed all available folders. This returned the INBOX, INBOX.Sent, INBOX.Drafts, and INBOX.Trash folders. The INBOX itself contained two emails:

->![](/images/2013-12-28/11.png)<-

I read the contents of both emails, however it was the second email that had some interesting information:

->![](/images/2013-12-28/12.png)<-

Here, the MySQL root password had been revealed. I attempted to use this password to login as the admin user in the forum, or the root user for FTP, but neither of them worked. There was another email in the INBOX.Sent folder, and this contained the root user's password for PHPMyAdmin:

->![](/images/2013-12-28/13.png)<-

I attempted to navigate to http://192.168.1.146/phpmyadmin but found nothing there. I figured that if there was something, then wfuzz would have found it earlier. After trying several other things, I was a bit lost. With the Christmas holidays approaching, I gave up for a few days to make merry, promising to come back to this again later. 

I looked over my notes again once Christmas was over, and realized that I had completely disregarded port 443. Out of curiosity, I punched in https://192.168.1.146/phpmyadmin and it sent me to a PHPMyAdmin login page!

->![](/images/2013-12-28/14.png)<-

Using the PHPMyAdmin password I found in mbrown's email, I was able to login and view the databases on the server: 

->![](/images/2013-12-28/15.png)<-

Determined not to miss anything again, I took my time exploring the contents of the databases and taking notes and screenshots. The exploration ended with me obtaining several password hashes. The first set was from the forum.mlf2_userdata, which were basically forum passwords and the email addresses associated with the user:

->![](/images/2013-12-28/16.png)<-

The next set of passwords were from the mail.admin and mail.mailbox tables which contained passwords for each user's IMAP/POP3 account: 

->![](/images/2013-12-28/17.png)<-

->![](/images/2013-12-28/18.png)<-

I ran each hash through Google and had no luck at all with the batch from the forum database. However, I was able to find the passwords for two of the accounts in the mail database: 

->![](/images/2013-12-28/19.png)<-

->![](/images/2013-12-28/20.png)<-

The passwords belong to users swillard and rhedley. I still had no SSH access, but I decided to give FTP another try. I wasn't able to login as swillard, but I was able to as rhedley. From there I was able to list all the home directories for each user on the server.

->![](/images/2013-12-28/21.png)<-

This user was able to view the contents of the incoming directory, which contained the file backup_webhost_130111.tar.gz.enc. 

->![](/images/2013-12-28/22.png)<-

I was able to download this file, but as the .enc extension suggested, it was probably encrypted and would do me no good until I knew what password was used to encrypt it.

I started exploring the home directories on the server. My first priority was to see if I could find an SSH key that I could use to get a proper shell on the server. In mbrown's .ssh directory, I found a file called downloadkey.

->![](/images/2013-12-28/23.png)<-

Examining this file showed that it was a private RSA key - possibly a copy of his id_rsa. Only one way to find out! I changed the file's permissions to read-only, and used it to connect to the target as user mbrown

->![](/images/2013-12-28/24.png)<-

I now had a proper shell on the server. After spending some time looking around, I came upon an interesting shell script that I had no read access to: /opt/backup.sh. Looking at the permissions, I noticed that it had an alternate form of access to it. Using getfacl, I discovered that the group ftpadmin had read access to this file.

->![](/images/2013-12-28/25.png)<-

A quick peek at /etc/group showed that rhedley and swillard both belonged to the ftpadmin group. User swillard also belonged to the sudo group, which may be handy later on in elevating to root privileges. Unfortunately I had no read access to any of the other user's home directories, except for sraines, and there was nothing in there other than an emtpy .bash_history file. 

I wondered if I could SSH into the server as a different user, so from mbrown's account, I attempted to SSH in as each user to see what would happen. As luck would have it, I managed to get a shell as swillard!

->![](/images/2013-12-28/26.png)<-

First thing I tried was sudo -l and provided the cracked password I found on Google Austin-Willard. This did not work. However, this user was part of the ftpadmin group, which meant I could read the contents of /opt/backup.sh

->![](/images/2013-12-28/27.png)<-

So a bit of a jackpot - the file contains the command and password used to encrypt the backup_webhost_130111.tar.gz.enc I downloaded earlier. Using the same command, I added the -d flag for decryption and saved the output as d.tar.gz

->![](/images/2013-12-28/28.png)<-

The decryption appeared to be successful. I extracted the contents of the archive and found that it was a backup of /etc. This treasure trove of files included a readable shadow file containing each user's hashed password!

->![](/images/2013-12-28/29.png)<-

Using unshadow, I merged the contents of passwd and shadow into a file called crack.me, which I could pass to john for cracking. I removed non-user accounts and ended up with the following:

```
sraines:$6$4S0pqZzV$t91VbUY8ActvkS3717wllrv8ExZO/ZSHDIakHmPCvwzedKt2qDRh7509Zhk45QkKEMYPPwP7PInpp6WAJYwvk1:1000:1000:Sandy Raines,401,1429,:/home/sraines:/bin/bash
mbrown:$6$DhcTFbl/$GcvUMLKvsybo4uXaS6Wx08rCdk6dPfYXASXzahAHlgy8A90PfwdoJXXyXZluw95aQeTGrjWF2zYPR0z2bX4p31:1001:1001:Mark Brown,404,2457,:/home/mbrown:/bin/bash
rhedley:$6$PpzRSzPO$0MhuP.G1pCB3Wc1zAzFSTSnOnEeuJm5kbXUGmlAwH2Jz1bFJU/.ZPwsheyyt4hrtMvZ/k6wT38hXYZcWY2ELV/:1002:1002:Richard Hedley,407,3412,:/home/rhedley:/bin/bash
```

This backup appeared to have been taken before sraines changed her account to swillard. However, if she didn't change her password, then I would be able to use it to run sudo with her swillard account. I fired up john, set it to use the darkc0de.lst wordlist on crack.me, and got up to make a coffee and watch TV for a few hours. 

It took about 3 hours, but finally, sraines' password was cracked:

->![](/images/2013-12-28/30.png)<-

Fingers crossed, I typed sudo -s, punched in the password, and got a root shell!

->![](/images/2013-12-28/31.png)<-

I checked the contents of /root to see if there was a flag in there. It contained a file called secret.jpg. I copied this over to /var/www and viewed the contents using my web browser.

->![](/images/2013-12-28/32.png)<-

Without any other information about the goal of this challenge, I assume this flag means I've completed it. I spent a bit of time checking for hidden messages in the JPG file but didn't find anything suspicious. Or I might have missed it. I don't know. 

I would say this is one of the more enjoyable De-ICE challenges I've attempted. Many thanks to [Hacking Dojo](http://hackingdojo.com/) for releasing this to the public. 
