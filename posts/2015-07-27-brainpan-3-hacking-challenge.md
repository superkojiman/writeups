---
layout: post
title: "Brainpan 3 hacking challenge"
date: 2015-07-27 00:15:37 -0400
comments: true
categories: boot2root
---

Brainpan 3 is finally here! If you've completed the previous Brainpan releases, then you'll know what to expect. This time round, I've made it a tad bit more challenging so get your caffeine shots ready!

<!--more-->

Brainpan 3 can be downloaded from VulnHub: [https://www.vulnhub.com/entry/brainpan-3,121/](https://www.vulnhub.com/entry/brainpan-3,121/)

It's also mirrored at [Google Drive](https://drive.google.com/open?id=0B41M3Dojh4xbUVczU09jWWtraW8), athough be warned that Google imposes bandwidth limits.

Here's the obligatory disclaimer: 

> By using this virtual machine, you agree that in no event will I be liable for any loss or damage including without limitation, indirect or consequential loss or damage, or any loss or damage whatsoever arising from loss of data or profits arising out of or in connection with the use of this software.

TL;DR: If something bad happens, it’s not my fault.

Brainpan 3 has been tested on VirtualBox, VMware Player, and VMware Fusion. Special thanks go out to my testers [barrebas](https://www.twitter.com/barrebas) and [Swappage](https://www.twitter.com/swappage). Drop them a note and thank them for helping me patch holes that would've made Brainpan 3 much easier.  

Import Brainpan_III.ova into your preferred hypervisor and configure the network settings to your needs. It will get an IP address via DHCP, but it’s recommended you run it within a NAT or visible to the host OS only since it is vulnerable to attacks.

As an added bonus, I'm giving away a Brainpan sticker to those who complete Brianpan 3 and publish their writeup. 

![](/images/2015-07-27/stickers.jpg)

I only have a handful of stickers, so to qualify for one, you need to: 

 * Attack Brainpan 3 remotely. No booting into single user mode, mounting the virtual disk externally, modifying the virtual machine, etc. 
 * Get root. Get the flag. 
 * Publish a writeup of how you totally pwned the challenge!
 * Message me on Twitter or email me to let me know you beat it. 

Be quick, because once they're gone, they're gone! Good luck!
