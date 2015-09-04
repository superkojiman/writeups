---
layout: post
title: "Creating a virtual machine hacking challenge"
date: 2013-12-10 18:10:26 -0400
comments: true
categories: howto boot2root
alias: /2013/12/creating-virtual-machine-hacking.html
---

After recently releasing the [Brainpan 2](/2013/11/19/brainpan-2-hacking-challenge/) hacking challenge, a handful of people asked me for tips on how to create their own hacking challenge. These virtual machine hacking challenges, more commonly known as boot2roots, are relatively easy to make, but cat be somewhat time consuming. In this post, I'd like to share some tips on how to roll out your own boot2root. 

<!--more-->

It begins with an idea; think of a challenge (or a set of challenges), that you'd like a player to solve. A goal should be set, such as obtaining root access on the virtual machine, or getting access to a flag. The challenges that you come up with, will define the difficulty level of the boot2root. In my experience, boot2roots geared towards beginners usually come with software that has a well documented vulnerability and exploit. More difficult ones require players to identify vulnerabilities in custom software, or to leverage difficult to find holes on the target, or a combination thereof. Keep in mind that the more difficult the challenge, the more time you'll be spending creating it and testing it. I think a good challenge is one that causes a bit of frustration to the player, but will in return provide a great sense of satisfaction once it's solved. 

Once you've come up with the challenge, you need to decide what operating system to host the challenge on, and what hypervisor to host the operating system on. Linux is a popular choice for several reasons; it's free and redistributable, you can create a fully operational server without paying a dime, you can keep it small which makes it more download-friendly, and there are so many different kinds to choose from. Obviously you should be careful about installing software that is not redistributable. Always check the license! Microsoft Windows operating systems are not redistributable, however there is a [way around that](http://blog.vulnhub.com/2013/02/introducing-vulninjector.html).

With regards to the hypervisor, I prefer creating my boot2roots using VMware ESXi and exporting it into an OVA file. OVAs can be imported by all incarnations of VMware, and by VirtualBox. I think it's important to make sure that your boot2root can be imported by the majority of people using hypervisors that are available to different platforms. Whatever hypervisor you choose, you should try to test it on the popular ones such as VMware Player and VirtualBox - both of which are free. 

Once you have your boot2root all packaged up, you should test it! Make sure your challenges can be solved, and that everything works as it should. You may find some bugs and have to redo your boot2root. I found this to be the most time consuming stage. In the end, I still missed a couple of things that made Brainpan 2 a wee bit easier, so it's a good idea to have someone else test your boot2root. A fresh set of eyes can help find some bugs. 

When you're satisfied with everything, it's time to make your boot2root public! Announce it on Twitter, your blog, IRC, whatever. [VulnHub](http://www.vulnhub.com/) makes a slew of boot2roots available so you should definitely consider having it hosted there. 

Hopefully this little guide will be useful to others thinking of creating their own boot2root. It's a time consuming process, but it's very rewarding to see others enjoying a good challenge that you created. 
