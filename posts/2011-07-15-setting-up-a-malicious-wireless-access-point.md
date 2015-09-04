---
layout: post
title: "Setting up a malicious wireless access point"
date: 2011-07-15 18:10:26 -0400
comments: true
categories: howto hacking
alias: /2011/07/setting-up-malicious-wireless-access.html
---

It can be tempting to hop onto an open wireless network when you just need to check your email, or you want to send off a tweet. Stop for a moment though, because an open wireless network might not be as safe as you think. With the right tools, an attacker can turn his laptop into an open wireless access point that captures your online activity. 

<!--more-->

By combining airbase-ng and sslstrip, we can turn our laptop into an access point that silently captures login credentials. This is a similar technique to what I posted before, except instead of connecting to a network and ARP poisoning the target, we lure the target into connecting to our network and sniff their activity.

You can grab the latest version of this script from [GitHub](https://github.com/superkojiman/peepshow).

Put your wireless card in monitor mode with airmon-ng and run the script. The script will create a logfile called log.txt which will contain all POST traffic. If you'd prefer to capture SSL and HTTP traffic, pass the -a option to sslstrip. By default the script creates an access point with an SSID of mylinksys. 

While the script is running, just tail the log file and you'll start to see entries such as this:

```
2011-07-15 18:20:27,752 SECURE POST Data (mlogin.yahoo.com):
_authurl=auth&_done=widget%3Aygo-mail%2Fhome.bp&_sig=&_src=
&_ts=1201692142&_crumb=fI0xXyxcgHakBrA2LoL9nA--&_pc=&_send_userhash=0&_appdata=&_partner_ts=&_is_ysid=1&_page=
secure&_next=nonssl&id=victim&password=hackme&__submit=Sign+In
```

Look at the last line and you'll see that we've captured the login ID for user victim with password hackme.

So the next time you see an open wireless access point, think twice before logging into it. This is but a simple passive attack that just involves gathering data. However, an attacker can setup a more aggressive attack which launches a set of exploits against each device that connects to the wireless access point in an attempt to gain access to the device itself.
