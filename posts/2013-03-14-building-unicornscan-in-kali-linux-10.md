---
layout: post
title: "Building Unicornscan in Kali Linux 1.0"
date: 2013-03-14 18:10:26 -0400
comments: true
categories: coding
alias: /2013/03/building-unicornscan-in-kali-linux-10.html
---

Unicornscan is no longer packaged with [Kali Linux 1.0](http://www.kali.org/). One of my scripts ([onetwopunch.sh](/2012/05/31/port-scanning-one-two-punch/)) happens to use it, so I went about building Unicornscan from source. I ran into a couple of snags when building it, but I've documented it here to make things easier for others.

<!--more-->

Unicornscan requires flex which isn't installed in Kali Linux 1.0, so install it:

```
root@kali# apt-get install flex
```

This should install smoothly, and once it's, download the latest version of Unicornscan from [http://www.unicornscan.org/](http://www.unicornscan.org/). Unpack it and build as follows: 

```
root@kali# ./configure CFLAGS=-D_GNU_SOURCE && make && make install
```

Once it's done, you'll find unicornscan in /usr/local/bin
