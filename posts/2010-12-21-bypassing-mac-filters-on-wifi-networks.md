---
layout: post
title: "Bypassing MAC filters on WiFi networks"
date: 2010-12-21 18:10:26 -0400
comments: true
categories: coding howto hacking
alias: /2010/12/bypassing-mac-filters-on-wifi-networks.html
---

Most wireless routers have a security feature called MAC filtering. Each network card on a computer comes with a unique MAC address. MAC filtering allows the user to specify which computers are allowed to use the wireless network by entering the computer's MAC address into the whitelist. This is a security tip that I see often when reading about securing wireless networks. When used by itself, or with WEP, it can give the user a false sense of security. I'm going to show you how this security layer can be bypassed.

<!--more--> 

For this hack, I've setup a test wireless network with a wireless router and three computers that are allowed to connect. A fourth computer, my laptop, will be used as the attacking computer. The wireless router's SSID is ghost, it uses WEP for encryption, and employs MAC filtering. Here's the attack plan:

1. Figure out what MAC addresses are on the whitelist so we know which computers are allowed to connect
1. Change our MAC address to one that's on the whitelist
1. Crack the WEP key
1. Use the WEP key and our fake MAC address to login to the wireless network

So the first step is to figure out what MAC addresses are on the whitelist. This is relatively easy and just involves running airodump-ng. Here's what it looks like when I point it at ghost:

```
 CH 11 ][ Elapsed: 2 mins ][ 2010-12-20 21:36
 
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
 
 00:22:6B:XX:XX:XX  -28  88     4522      196    1  11  54 . OPN              ghost
 
 BSSID              STATION            PWR   Rate    Lost  Packets  Probes
 
 00:22:6B:XX:XX:XX  00:21:19:XX:XX:XX   -9    1 -24      0        4
 00:22:6B:XX:XX:XX  00:1D:4F:XX:XX:XX  -38   54 -54      0       23
 00:22:6B:XX:XX:XX  00:26:BB:XX:XX:XX   -1    1 - 0      0        1
```

Right away we see that there are three clients connected to ghost. More importantly, it displays their MAC addresses under the STATION column. Now we know which computers are on the whitelist. 

Next, we need to trick the wireless router into allowing our computer to join the network. The way we do this is by changing our MAC address to one that's on the whitelist. Let's target 00:1D:4F:XX:XX:XX. The series of commands that will change our MAC address:

```
ifconfig mon0 down
macchanger -m 00:1D:4F:XX:XX:XX mon0
ifconfig mon0 up
```

At this point we can start cracking the WEP key using a combination of airodump-ng, aireplay-ng, and aircrack-ng. There are also various automated tools to assist with this, such as the script I described in a previous [post](/2010/04/24/automating-the-checkmate-of-wifi-wep-networks/)

Start by capturing all the packets:

```
airodump-ng -c 11 -d 00:22:6B:XX:XX:XX -w dump mon0
```

For this example I'm using the ARP request replay attack. You may need to experiment with the different kinds of attacks until you get one that works properly:

```
aireplay-ng -3 -b 00:22:6B:XX:XX:XX mon0
```

Finally, crack the WEP key: 

```
aircrack-ng dump*.cap
```

Assuming all of that went well, you should now have the WEP key required to authenticate to the wireless network. 

Final step, authenticate to the wireless network. I'm using a USB based wireless adapter, so I just unplug it and plug it back in, then change the MAC address again to one that's on the whitelist and use the network management program to authenticate and log into the wireless network. So we've seen how easy it is to bypass this security layer. That doesn't mean you shouldn't use it though, every extra hurdle the attacker has to go through puts them one step further from their goal. While it won't stop a determined attacker, it will stop casual users and script kiddies. If you choose to use MAC filtering, pair it with WPA2 and use a good passphrase to keep out the bad guys.
