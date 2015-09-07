---
layout: post
title: "Sniffing website login credentials"
date: 2011-07-13 18:10:26 -0400
comments: true
categories: howto hacking
alias: /2011/07/sniffing-website-login-credentials.html
---

Man-in-the-middle (MITM) attacks are an effective way to capture data flowing between a target and the router. In a nutshell, the attacker places himself between the target and the router so that all data flows through the attacker's machine. The target thinks he's communicating with the router, and the router thinks it's communicating with the target, when in reality, they are communicating with the attacker and the attacker just relays the information back and forth. It's like a malicious mailman who reads your letters before sealing them and sending them off. 

<!--more-->

Using a combination of tools readily available online such as arpspoof, ettercap, and sslsniff, we can capture a user's login credentials and trick them into thinking that they are logging in through an encrypted connection.

We'll use arpspoof to poison the ARP cache between the target and the router. This tells the router that we're the target, and tells the target that we're the router. Next we setup firewall rules to redirect connections to port 80 (HTTP) to port 10000 (the port sslstrip will be listening on). Next, run sslstrip to wait for incoming connections. sslstrip will receive the target's requests to login to the server and will create its own SSL connection with the server and relay the information back and forth. The end result is the server thinking everything is encrypted and fine, and the target not having an encrypted connection at all. Finally, run ettercap to sniff for incoming connections and print the cleartext login credentials to the screen.

Some changes need to be made to the etter.conf file for this to work, specifically the ec_uid, ec_gid, redir_command_on, and redir_command_off. Update them accordingly so they look like this:

```
ec_uid = 0                # nobody is the default
ec_gid = 0                # nobody is the default
 
redir_command_on = "iptables -t nat -A PREROUTING -i %iface -p tcp --dport %port -j REDIRECT --to-port %rport"
redir_command_off = "iptables -t nat -D PREROUTING -i %iface -p tcp --dport %port -j REDIRECT --to-port %rport
```

You can also create an alternative etter.conf file with these changes if you don't want to mess with your existing etter.conf. You can pass the alternative configuration to ettercap using the -a flag. 

The script that combines all the commands necessary to setup the attack can be downloaded from [GitHub](https://github.com/superkojiman/snuff). I'm using BackTrack Linux 4RT2 for this setup. 


Wait for the target to navigate to HTTP login sites and enter their credentials and watch ettercap print them to the screen. To quit the script, just hit CTRL-C and it will cleanup after itself. 
