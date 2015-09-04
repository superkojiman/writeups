---
layout: post
title: "The Offensive Security Playground: A Review"
date: 2014-12-05 20:19:19 -0500
comments: true
categories: musings
---

A couple of years ago I successfully completed the Offensive Security [Pentesting with Backtrack](http://www.offensive-security.com/information-security-training/penetration-testing-with-kali-linux/) (PWB) course, and a year after that, the [Cracking the Perimeter](http://www.offensive-security.com/information-security-training/cracking-the-perimeter/) (CTP) course. Having a huge lab made up of different machines in different subnets to break into is just a great challenge. When I completed the courses, I was a little sadenned that I'd no longer get a chance to poke at the labs. So you can imagine my excitement when I was asked if I'd like to beta test Offensive Security's latest offering; [The Playground](http://www.offensive-security.com/offensive-security-solutions/virtual-penetration-testing-labs/). 

<!--more-->

->![](/images/2014-12-05/01.jpg)<-

First a bit of background about myself. I'm not a professional pentester, nor do I work in infosec. I just do this for fun. It's a hobby, and one that I enjoy. I took PWB and CTP because I thought it'd be a great way to test myself. That said, I was excited to return to the Offensive Security labs, and so I readily agreed to beta test the playground.

I received an email containing a connectivity pack to VPN into the playground, instructions, and access to the playground's dashboard. Offensive Security recommended using a 32-bit Kali Linux VM for this, so I downloaded one and customized it to my liking. The beta testing was initially just for two weeks, but it was generously extended by a few more days later on. Once I had connected to the playground via VPN, the first thing I did was to check out the dashboard. 

->![](/images/2014-12-05/02.jpg)<-

The dashboard contains a listing of all the machines that can be attacked. One of the first things I noticed was that it listed machines in different networks. It looked like some of the other machines were in internal networks that I would eventually need to pivot to in order to reach them. The dashboard also allowed me to revert and reboot each machine. Each machine has an original state and reverting it brings it back to this state. It's handy because the lab is shared by other attackers, so in order to ensure that you're attacking a machine in a clean state, you need to revert it. 

The other feature in the dashboard is the ability to enter an MD5 hash for each machine. Each machine has a file called proof.txt which contains a flag in the form of an MD5 hash. Entering this flag into the dashboard will mark that machine as completed, and give you a point.

After a week of beta testing, I had already encountered various Linux distributions, some Windows machines, and lots of different vulnerabilities, ranging from ridiculously easy to hair pulling. I had even managed to get a foothold onto one of the internal networks and hacked into some of the machines in there. As I popped each machine, I documented the steps for my own records, although I imagine once the playground goes into production you may be expected to write a proper report. 

Was it everything I expected it to be? Yes, and much more. To avoid spoilers, I won't be giving out specifics on the targets and the networks. These can change anyway before the playground goes live. 

There are machines in the playground that cater to different experience levels so even a beginner can score some points. Some machines even have defenses that can get you blocked temporarily if you prod them wrong. While a beginner can get at the low hanging fruit in the starting network, more knowledge is required to attack the more difficult machines and to pivot into the internal networks. Furthermore, machines in the playground don't exist within their own bubble. Much like a real network, information looted from a compromised machine may be used to gain access to other machines that would otherwise be impossible to break into. 

Overall, the playground provides a great environment for learning and experimenting without worrying about breaking anything. At the time of writing, I still have a few days left before my testing time is over, so I hope to get as deep into the network as I possibly can. 

There were some things however that I wasn't overly fond of. The ability to revert a machine is essential before attacking it in order to ensure that you're starting from a clean slate. However, since the playground is shared by other attackers, there's a good chance that someone will revert a machine you're currently working on. This is particularly painful when someone reverts a machine used as a pivot point into an internal network. Having your already slow port scan die halfway through because someone reverted the pivot point is a whole other kind of frustration. 

I'm not sure if this will change in the future, but at the moment, each machine only nets you 1 point despite its difficulty level. If you've played in Capture The Flag competitions, you'll know that the more difficult the challenge, the greater the points. Having a score listed beside each target in the dashboard would have allowed me to quickly attack the easy machines first to get them out of the way before focusing my efforts on the more difficult targets. This would also be handy for beginners. At the same time, I understand that in a real pentesting engagement you won't know which targets are easy and which are difficult. Perhaps I'm just looking at this more from a gaming perspective. 

What if you've already completed PWB/PWK? The playground consists of entirely different machines and different networks, most of which you won't see in the PWB/PWK labs. To quote from Offensive Security's website: 

> Our playground systems include Citrix environments, Windows Active Directory Domains, SCADA networks, IPS systems and corporate Antivirus solutions – all adding dimensions of difficulty and realism to the labs.

So you'll definitely be up against some new targets, but you'll feel right at home. Everything you've learned in PWB/PWK can be applied to the playground. In a way, the playground makes for a perfect extension to PWB/PWK. 

To summarize, my experience with the playground was mostly positive, and extremely enjoyable. If you're looking for a way to test your skills against different targets and configurations, the playground is a good place to do so. According to Offensive Security's website, the playground is planned to go live in early 2015. You can get more details from [http://www.offensive-security.com/offensive-security-solutions/virtual-penetration-testing-labs/](http://www.offensive-security.com/offensive-security-solutions/virtual-penetration-testing-labs/). 

In closing, I want to express my appreciation to Offensive Security, particularly [muts](https://twitter.com/kalilinux) and [g0tmi1k](https://twitter.com/g0tmi1k), for giving me the opportunity to beta test the playground. There was pain, frustration, and that high you experience after rooting a machine that had been driving you crazy for hours. Thank you!
