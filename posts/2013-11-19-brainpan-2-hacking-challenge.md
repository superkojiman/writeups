
When I initially created [Brainpan](/2013/03/20/brainpan-hacking-challenge/), my intent was to give back to the community with something fun and challenging. It didn't occur to me that others would find it so enjoyable that they would want more. I had a blast creating the first challenge, and so I thought, "What the hey, let's create a second one!". And so I present, Brainpan 2. Your goal is to break into the server and read the contents of /root/flag.txt 

<!--more-->

To top it off, [VulnHub](http://www.vulnhub.com/) has been kind enough to not only host the virtual machine, but to award a prize for the best written solution to Brainpan 2! The contest begins on November 20th, and ends on December 4th. For further details, head over to [http://blog.vulnhub.com/2013/11/competition-brainpan-2.html](http://blog.vulnhub.com/2013/11/competition-brainpan-2.html)

You can download Brainpan 2 here: [http://vulnhub.com/entry/brainpan_2,56/](http://vulnhub.com/entry/brainpan_2,56/)

It is also mirrored at [Google Drive](https://drive.google.com/file/d/0B41M3Dojh4xbRDREVUtySTBHZGs/edit?usp=sharing), although be warned that Google imposes bandwidth limits. 

Now as before, the obligatory disclaimer:

>By using this virtual machine, you agree that in no event will I be liable for any loss or damage including without limitation, indirect or consequential loss or damage, or any loss or damage whatsoever arising from loss of data or profits arising out of or in connection with the use of this software. 

TL;DR: If something bad happens, it's not my fault.

Brainpan 2 has been tested and found to work on the following hypervisors: 

* VMware Player 6.0.1
* VMWare Fusion 6.0.2
* VirtualBox 4.3.2

Import brainpan2.ova into your preferred hypervisor and configure the network settings to your needs. It will get an IP address via DHCP, but it's recommended you run it within a NAT or visible to the host OS only since it is vulnerable to attacks.
