---
layout: post
title: "Fight scareware: Learn how your anti-virus works"
date: 2010-03-12
comments: true
categories: musings
alias: /2010/03/fight-scareware-learn-how-your-anti.html
---

[Wikipedia](http://en.wikipedia.org/wiki/Scareware) defines scareware as:

>A tactic frequently used by criminals involving convincing users that a virus has infected their computer, then suggesting that they download (and pay for) antivirus software to remove it. Usually the virus is entirely fictional and the software is non-functional or malware itself.

<!--more-->

Scareware usually hits you when you're browsing a website that's controlled by a
malicious user. By taking advantage of [search engine optimization poisoning](http://isc.sans.org/diary.html?storyid=8098) search engine optimization poisoning techniques, a Google search for a hot topic might actually redirect you to a malicious site. I won't talk about SEO poisoning here, suffice it to say that an attacker can influence Google results so that his website shows up at the top of the list. This tactic is effective when people search for newly announced technologies, celebrity deaths, or natural disasters. Why should you be concerned? Because someday, searching for "new iphone 4G" might actually send you to a malicious website.

Scareware is designed to look like legitimate anti-virus software, usually a Windows based one. You'll see a popup window with what appears to be the contents of your hard drive, a moving progress bar, and warning messages informing you that your computer is infected. You'll be given an option to download the software or cancel the popup. I should mention that in most cases, either option will download the software.

I believe one reason scareware is so effective is that the majority of computer users don't know what happens when their anti-virus detects malware. Almost everyone has an anti-virus solution from some well-known company, yet only a few have actually seen what happens when it goes into red-alert. The chances of being tricked by malware becomes very slim when you know what should happen when your anti-virus kicks in. So how do you test your anti-virus? Well you could infect it with a real virus, but that would be kind of irresponsible.

The solution is in the [EICAR](http://www.eicar.com) (European Institute for Computer
Antivirus Research) test. The EICAR test is designed so that anti-virus companies and
users can test to see if the anti-virus works. The test consists of downloading or
creating a test file containing a specific set of characters designed to trigger any
anti-virus to go off. The file itself is <b>not</b> a virus. EICAR offers several
variants of the that can be downloaded here:
[http://www.eicar.org/anti_virus_test_file.htm](http://www.eicar.org/anti_virus_test_file.htm)

Any decent anti-virus scanner will immediately flag this as a virus the moment you download it. That is, before you even run it, it should be flagged and quarantined by your anti-virus. Take note of how your anti-virus warns you of the EICAR test file. This is what you need to expect when your computer is infected with a real virus. If at some point in the future you get a virus infection warning that looks nothing like what you just saw, then it's probably scareware.

Anti-virus companies have gone to great lengths to keep their tools user-friendly, so taking a couple of minutes to see how it works is well worth the effort.

For further reading:

[EICAR test file on Wikipeedia](http://en.wikipedia.org/wiki/EICAR_test_file)

[Symantec SEO poisoning article](http://www.symantec.com/connect/blogs/iframes-please-make-way-seo-poisoning)

[The register article on the booming scareware business market](http://www.theregister.co.uk/2009/08/07/scareware_market/)
