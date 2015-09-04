---
layout: post
title: "Open, Simsim!"
date: 2009-10-12 20:27:05 -0400
comments: true
categories: hacking coding
alias: /2009/10/open-simsim_12.html
---

There've been a few times where I've found myself in need of a default username/password for a wireless router that has been factory reset, or was just improperly configured. Going through manufacturer websites searching for the manuals and the information can be a pain. 

<!--more-->

Fortunately there are already a few resources online that provide lists of default credentials for various devices. So why not just use that? Well, having a local copy is nice because sometimes I may not have Internet access, or I may want to format the results in a different way so I can pipe it into another program.

That's where opensesame comes in. It's a python script that queries a sqlite3 database containing the default credentials for certain devices. This database was created using the publicly available default password list at [http://www.phenoelit-us.org/dpl/dpl.html](http://www.phenoelit-us.org/dpl/dpl.html)

opensesame will search the database looking for parameters specifed by the user such as vendor, model or both and displays the results. For example, searching for the default credentials for a Linksys WRT54G router yields several results, one of which is:

```
Vendor: linksys
Model: wrt54g
Version: 
Access Type: Multi
Username: admin
Password: admin
Privileges: Admin
Notes: Router / VoIP Gateway (@ 192.168.3.1)
```

The included README file contains more examples and information.

Download [OpenSesame 0.1](http://www.techorganic.com/software/opensesame/opensesame-0.1.tar.gz)


MD5   : 3f72ce1196510a71794600fe7504ab4a

SHA-1 : 8230c06127879e20deca8bbaeed025bf3ada4bf7

GnuPG signature: [opensesame-0.1.tar.gz.sig](http://www.techorganic.com/software/opensesame/opensesame-0.1.tar.gz.sig)
