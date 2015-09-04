---
layout: post
title: "One triple grande non-fat wireless hack please!"
date: 2010-04-15
comments: true
categories: musings
alias: /2010/04/one-triple-grande-non-fat-wireless-hack.html
---

Coffee shops can make an excellent location for attacking wireless networks. If a hacker can pick up your access point's signal, they can break into it from a coffee shop, the parking lot, or across the street. Most coffee shops such as Starbucks are popular hang outs for students with laptops, allowing the hacker to easily blend in without attracting suspicion. And of course there's the coffee and a comfortable place to sit.

<!--more-->

Wireless networks operated at home are notoriously weak. The majority of home wireless networks run off wireless modems provided by an ISP, or a wireless router purchased from a store. These wireless access points tend to have poorly configured factory settings. While they do offer an interface for the user to harden their security, most users don't know anything about wireless security and are just interested in getting the wireless access working. To top it off, these access points usually come with little or no logging functionality which makes it difficult to detect the presence of a hacker.

If you can find a coffee shop located close to an apartment (and there are plenty of them), there's a good chance that you'll be able to find several access points that can be easily cracked. 

If you're still using WEP, it's as good as having an open wireless network. WEP can be cracked in a couple of minutes. WPA/WPA2 is a step up, but if you use a poor password, a dictionary attack can crack it in no time. Hiding the SSID or employing MAC address filtering provide a false sense of security. The SSID can be easily de-cloaked, and whitelisted MAC addresses can be easily captured and spoofed.

How hard is it to crack into a wireless network? Ridiculously easy and cheap. For
hardware, a decent [wireless adapter](http://www.dlink.com/products/?pid=334) capable of
packet re-injection costs about USD $20.00 on eBay. An even better
[one](http://www.data-alliance.net/-strse-73/Alfa-AWUS036H-500mW-USB/Detail.bok costs
about $40. For software, [Backtrack Linux](http://www.backtrack-linux.org/) is a free
Linux distribution geared towards penetration testing. It contains everything a hacker
needs to successfully compromise wireless networks. There are excellent [tutorials](http://www.aircrack-ng.org/doku.php?id=tutorial) and resources available online on how to use these tools. 

So how do you protect yourself against these attacks? The best security available right now for home wireless networks is to use WPA2 with AES, and a strong passphrase. This configuration takes under 5 minutes to set up, and is well worth the effort. 

It's also a good idea to change the passphrase every month or so because although a good passphrase will most likely survive a dictionary attack, a brute force attack will crack it in time. Fortunately, brute force attacks on a strong passphrase can take several years. This makes it an impractical attack for hackers especially if the passphrase is changed at the end of each month. By the time the passphrase is cracked, it is no longer applicable.

Attackers know the weaknesses in wireless networks. Making your network a less attractive target is an effective way to force most attackers to move on to lesser protected targets.
