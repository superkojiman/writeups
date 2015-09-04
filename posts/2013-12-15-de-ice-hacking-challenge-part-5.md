---
layout: post
title: "De-ICE hacking challenge: Part 5"
date: 2013-12-15 18:10:26 -0400
comments: true
categories: boot2root
alias: /2013/12/de-ice-hacking-challenge-part-5.html
---

This is a walkthrough for the De-ICE S1.120-1 B challenge, which can be downloaded here: [http://vulnhub.com/entry/de-ice_s1120-b,11/](http://vulnhub.com/entry/de-ice_s1120-b,11/). The author describes this challenge as "moderately difficult". Itching for a good challenge, I decided to see if it lived up to its difficulty level. 

<!--more-->

Using netdiscover, I identified the target's IP address as 192.168.1.20. I started off by running a portscan against it.

![](/images/2013-12-15/01.png)

A good handful of open TCP ports were detected. I started off by browsing to the website. This gives a brief description about the challenge, and a link to the actual webpage for the challenge:

![](/images/2013-12-15/02.png)

This page contained some information that could be used for enumerating, namely a contact address and a phone number. 

I ran nikto against this website and it uncovered a info.php page, which identified the target as running Linux 2.6.16, among other things. 

![](/images/2013-12-15/03.png)

Thinking that there might be some hidden directories, I ran wfuzz on the target, but it didn't find anything. At this point, I decided to move on to the next service, FTP on port 21. 

Anonymous FTP was allowed, however the service itself seemed to be broken. After logging in, the server would throw an error whenever I entered a command.

![](/images/2013-12-15/04.png)

This seemed to be another dead end. The other services, with the exception of SMTP, would require user credentials in order make use of them. I looked around for vulnerabilities affecting the other services but found nothing that would help me get a shell. 

From the webpage, I knew there was a customerserviceadmin user, so I decided to see if that was a valid user. The easiest way to do that was through SMTP.

I created a file called smtp.txt containing the following users:

```
customerserviceadmin
root
superkojiman
```

My own username was added to the list purely as a control element, as will become clear shortly. For enumerating users in SMTP, I used smtp-user-enum. I started off by using VRFY to check for users:

![](/images/2013-12-15/05.png)

smtp-user-enum claims that all users exist, which couldn't be true as I was fairly certain there was no superkojiman user account on the server. This is what I meant by using my username as a control element. The VRFY method obviously didn't work, so I tried RCPT next:

![](/images/2013-12-15/06.png)

Much better, this time only root was detected. However customerserviceadmin was not detected, which implied that user did not exist on the target, or a different username was assigned to the Customer Service Admin account. [Previously](/2013/12/13/de-ice-hacking-challenge-part-4), I had obtained a list of usernames from another De-ICE challenge, so I thought I would try that. Unfortunately, it returned no results. I decided to try different variations of customerserviceadmin instead. Off the top of my head, I made a quick list: 

```
custserviceadmin
custservadmin
custservadm
customerserviceadm
customerservadm
customersa
customerservicea
customerserva
custserva
cserviceadmin
cserviceadm
cservadmin
cservadm
csadmin
csadm
csa
```

Fingers crossed, I ran this against smtp-user-enum once again, and got a hit!

![](/images/2013-12-15/07.png)

The account csadmin was discovered on the target. With nothing else to go on, I had to resort to a brute force attack. I selected a wordlist containing the top 1000 popular passwords knowing that it would finish relatively quickly and if csadmin's password was simple, it was likely to be in there. Sure enough, the password for csadmin was found in a matter of minutes: 

![](/images/2013-12-15/08.png)

I used the password to login to the target and found two files in csadmin's home directory:

![](/images/2013-12-15/09.png)

The file mailserv_download/2010122014234.j12Gqo4H049241 was an email from the sdadmin user account to the csadmin user. The email was an invitation to sdadmin's son's birthday party:

```
To: csadmin@nosecbank.com
CC: 
Subject: My Son's Birthday
Date: Mon, 20 Dec 2010 14:23:46 +0500
Return-Path: <sdadmin@nosecbank.com>
Delivered-To: csadmin:nosecbank.com@nosecbank.com
Received: (qmail 20281 invoked from network); 20 Dec 2010 09:23:46 -0000
X-Received: from network (192.168.1.123) by mailserv1-3.us6.service.com; 
20 Dec 2010 09:23:46 -0000
Received: from www.nosecbank.com (unknown [198.65.139.34]) by 
srv5.us6.service.com (Postfix) with ESMTP id D98402459DD for 
<csadmin@nosecbank.com>; Mon, 20 Dec 2010 09:23:46 +0000 (GMT)
Message-Id: <2010122014234 data-blogger-escaped-.j12gqo4h049241="" data-blogger-escaped-www.nosecbank.com="">
Mime-Version: 1.0
Content-Type: multipart/alternative; 
boundary="---=_NextPart_000_0000_02F24S11.FEPQRE80"
X-Mailer: K-Mail; Build 1.0.5510
Thread-Index: Qw2cWVmE3odZs3TqTTqFvS1e3lexms==
Message: Hey Mark, I am curious if you would be free to come over and 
visit for my son Donovin's birthday tomorrow after work.  I would also 
appreciate if you brought Andy with you as well, because Donny 
really enjoyed playing with him last time he was over.  I know its short 
notice but he is turning 12 and it is special for both him and me. Let 
me know if this works. Thanks!  -Paul
<!--2010122014234-->
```

The second file, 2010122216451.f81Ltw4R010211.part2 appeared to be some binary file that contained some bits of code:

![](/images/2013-12-15/10.png)

At this point I wasn't aware of what the code was for, so I went ahead and explored the server for more clues. 

I found three other users on the server, sdadmin, sysadmin, and dbadmin. I also discovered an encrypted file in /home/ftp/incoming called useracc_update.csv.enc

![](/images/2013-12-15/11.png)

In order to decrypt this, I would need to know the cipher used to encrypt it, and the password. This is very much like the first [De-ICE Level 1](/2011/07/19/de-ice-hacking-challenge-part-1/) challenge. 

After looking around and finding nothing to elevate my privileges, I decided to take a closer look at the email in csadmin's home directory. Several bits of information could be gathered from this email:

Paul has a son Donovin
Donovin's birthday is tomorrow. The email was sent on Dec 20 2010, which means his birthday is Dec 21
Donovin is 12 years old, which makes the year he was born 1998

I thought this information might be useful in guessing the password for the sdadmin account. To do this, I used the Common User Password Profiler script. This didn't seem to be available in Kali Linux by default, so I just grabbed it from [http://www.remote-exploit.org/articles/misc_research__amp_code/index.html](http://www.remote-exploit.org/articles/misc_research__amp_code/index.html). I launched the script and entered whatever information I knew:

![](/images/2013-12-15/12.png)

5,486 potential passwords were generated. Once again, it's back to hydra for more password guessing. This time, using sdadmin.txt as the wordlist. It took a bit longer than the first previous time, but it identified the password for sdadmin as donovin1998:

![](/images/2013-12-15/13.png)

I used the newly obtained password to login as sdadmin. Much like the csadmin account, the sdadmin user also has two similar files; an email and a data dump containing some code fragments. 

![](/images/2013-12-15/14.png)

I had a feeling the email would probably contain a clue that would allow me to elevate my privileges or switch to another user account:

```
To: sdadmin@nosecbank.com
CC: 
Subject: RE: My Son's Birthday
Date: Mon, 20 Dec 2010 15:04:32 +0500
Return-Path: <csadmin@nosecbank.com>
Delived-To: sdadmin:nosecbank.com@nosecbank.com
Received: (qmail 20281 invoked from network); 20 Dec 2010 10:04:32 -0000
X-Received: from network (192.168.1.123) by mailserv3-4.us6.service.com; 
20 Dec 2010 10:04:32 -0000
Received: from www.nosecbank.com (unknown [198.65.139.32]) by 
srv3.us6.service.com (Postfix) with ESMTP id D98214787FD for 
<csadmin@nosecbank.com; Mon, 20 Dec 2010 10:04:32 +0000 (GMT)
Message-Id: <2010122015043 data-blogger-escaped-.j15htu1h341102="" data-blogger-escaped-www.nosecbank.com="">
Mime-Version: 1.0
Content-Type: multipart/alternative; 
boundary="---=_NextPart_000_0000_05F11S20.FGHZWE49"
X-Mailer: K-Mail; Build 1.0.5510
Thread-Index: Aa5fqAwT8nsBe3T3T5q67a3Fd22XsZ==
Message: Hey there Paul!  I would gladly bring myself and my son over 
tomorrow after work!  I would only be hesitant to come over if you 
invited Fred over too... He just freaks me out sometimes.  It doesnt 
help that he locks himself up in his office and is anti-social during 
lunch hours... On top of that he calls himself the "databaser"!  I mean, 
who in thier right mind does that?  Either way, I will most likely be 
there tomorrow.  I look forward to it!   -Mark
<!--2010122015043-->
```

The password "databaser" seemed to go hand-in-hand with the dbadmin user. Of course it didn't work when I tried it, which meant there was probably more brute forcing required. Once again, I fired up cupp.py and entered some information about the dbadmin user that I obtained from the email:

![](/images/2013-12-15/15.png)

This time it generated 2,568 possible passwords. I fired up hydra and after a few minutes, a password for dbadmin was found:

![](/images/2013-12-15/16.png)

I logged in to the server as dbadmin using the password databaser60. Unlike the other two accounts, this one only had a data file containing code fragments. Each of the code fragments I'd found in all three accounts had some identifying name to them, namely, part 1, part 2, and part 3. Based on the information in part 3, this program was used to create the passwords for the root and sysadmin accounts. If I could piece it back together, I should be able to use it to get those passwords. 

Using the strings command, I was able to quickly view the relevant parts that I needed to copy. The code appeared to be Java, so I put them into a Decoder.java file. I added a main() function, fixed a couple of compilation errors, and ended up with the following:

```java
public class Decoder {
 // PART 1
 
 int[] processLoop(String input){ 
  int strL=input.length();
  int lChar=(int)input.charAt(strL-1); 
  int fChar=(int)input.charAt(0); 
  int[] encArr = new int[strL+2]; 
  for(int i=1;i < strL+1;i++){ 
   encArr[i]=(int)input.charAt(i-1);} 
  encArr[0]=(int)lChar; 
  encArr[encArr.length-1]=(int)fChar; 
  encArr=backLoop(encArr); 
  encArr=loopBack(encArr); 
  encArr=loopProcess(encArr); 
  int j=encArr.length-1; 
  for(int i=0;i < encArr.length;i++){ 
   if(i==j) break;
   int t=encArr[i]; 
   encArr[i]=encArr[j]; 
   encArr[j]=t;j--;}
  return encArr;} 
 
 /*
  * Note the pseudocode will be implemented with the  
  * root account and my account, we still need to implement it with the csadmin, sdadmin, 
  * and dbadmin accounts though
  */
 
 // PART 2
 int[] backLoop(int[] input){ 
  int ref=input.length;
  int a=input[1]; int b=input[ref-1]; 
  int check=(a+b)/2; 
  for(int i=0;i < ref;i++){ 
   if(i%2==0) input[i]=(input[i]%check)+(ref+i); 
   else input[i]=(input[i]+ref+i);}
  return input;} 
 int[] loopProcess(int[] input){  
  for(int i=0;i < input.length;i++){ 
   if(input[i]==40||input[i]==41) input[i]+=input.length; 
   else if(input[i]==45) input[i]+=20+i;} 
  return input;} 
 
 
 // PART 3
 int[] loopBack(int[] input){ 
  int ref=input.length/2; 
  int[] encNew =new int[input.length+ref]; 
  int ch=0; 
  for(int i=(ref/2);i < input.length;i++){ 
   encNew[i]=input[ch]; ch++;} 
  for(int i=0;i < encNew.length;i++){ 
   if(encNew[i]<=33) encNew[i]=33+(++ref*2); 
   else if(encNew[i] >= 126) encNew[i]=126-(--ref*2); 
   else{ 
    if(i%2==0) encNew[i]-=(i%3); 
    else encNew[i]+=(i%2);}} 
  return encNew;}
 
 
 public static void main(String[] args) {
  Decoder d = new Decoder();
  int[] c = d.processLoop(args[0]); 
  for (int i = 0; i < c.length; i++)
   System.out.print((char)c[i]); 
 }
}
```

I compiled this and ran it to get the passwords for sysadmin and root. 

![](/images/2013-12-15/17.png)

I attempted to login as root over SSH, but that didn't work out. Perhaps root login over SSH was disabled, or the password was incorrect. I decided to try logging in as sysadmin next with the password {% raw %}
7531/{{tor/rv/A
{% endraw %} and was successful. Once I had a shell, I used su and the root password to get a root shell:

![](/images/2013-12-15/18.png)

Game over? Not quite. I noticed that the sysadmin user had a file called Note_to_self. The contents state that the useracc_update.csv had been moved to the FTP server and encrypted for the dbadmin user. My initial assumption was to use the password databaser60 again, but first I had to figure out the cipher used to encrypt the file. 

To do this, I just cycled through all the supported OpenSSL ciphers and saved the decrypted contents to a file, out.txt

![](/images/2013-12-15/19.png)

I then ran out.txt through the strings command and piped everything to less. The idea was that if the password was correct, somewhere in that file would be some text that wasn't gibberish:

![](/images/2013-12-15/20.png)

Unfortunately there was nothing that looked like decrypted text. It was likely that the password I was using was incorrect. Perhaps the root user's password was the correct one? I tried it again, using the password 31/Fwxw+2 and this time, found something interesting in the out.txt:

![](/images/2013-12-15/21.png)

Using the aes-256-cbc cipher and the root password, the encrypted text was successfully decrypted revealing the user account information. Finally, game over at this point. 

This challenge seemed to focus more on brute force attacks and password guessing. For beginners, this challenge would be a good exercise in information gathering techniques and creating and using different wordlists. With regards to difficulty, it's not that much more difficult than the other De-ICE challenges, but it does require a bit of programming knowledge to fix-up the scattered code fragments on the server. 
