
There are legitimate reasons for wanting to stay anonymous online. You don't have to be living in an oppressed country, or be a criminal, or an activist. Sometimes you just don't want Facebook or Twitter to know where you're connecting from.

<!--more-->

A popular service for anonymous browsing is [Tor](https://www.torproject.org/) (The Onion Router). When you browse the Internet with Tor, your activity goes through various servers with the intention of anonymizing your location. So if you live in Australia, and you're browsing through Tor, your target website may see that you are coming from an IP address from Germany, or even some other part of Australia - but never your actual IP address. Tor also has the added benefit of being able to access some services restricted only to certain countries.

Setting up Tor is relatively easy, in fact if all you want to do is stay anonymous when you browse online, just install the [Tor Browser Bundle](https://www.torproject.org/download/download.html.en) and you're done. You can even copy it into a USB flash drive so you can carry it with you and use it wherever you go. If you want more flexibility, I would suggest installing the Vidalia Bundle instead.

The Vidalia Bundle packages Tor and the Polipo proxy, but does not include the Firefox browser. When you launch Vidalia, it sets up the Tor circuits and opens an HTTP proxy on port 8118, and a SOCKS proxy on port 9050. The Vidalia Bundle also comes with the Torbutton extension for Firefox. When you install the Torbutton extension, you will have the ability to quickly switch Firefox between Tor browsing, and regular browsing.

I mentioned that the Vidalia Bundle offers more flexibility. Since Vidalia opens an HTTP proxy on port 8118 and a SOCKS proxy on port 9050, you can configure any application that has proxy or SOCKS support to point to one of these ports so that it can take advantage of the Tor network. For example, let's say you're using an instant messenger like ICQ. You can go into it's setup and tell it to use a SOCKS proxy listening on your machine on port 9050, and now it will go through the Tor network.

What about applications that don't support proxies? Let's say you need to use telnet or mysql terminal commands. One solution is to use proxychains. If you run Linux or Mac OS X, you can grab proxychains 4 from [https://github.com/haad/proxychains](https://github.com/haad/proxychains). The default configuration works off the bat and you can start using proxychains right away. To use it, just call proxychains followed by the command you want to anonymize. For example, here's my IP address without using proxychains:

```
$ wget --quiet -O - http://whatismyip.akamai.com/
128.100.XXX.XXX
```

And here it is with proxychains:

```
$ proxychains4 wget --quiet -O - http://whatismyip.akamai.com/
[proxychains] config file found: /usr/local/etc/proxychains.conf
[proxychains] preloading /usr/local/lib/libproxychains4.dylib
[proxychains] DLL init
[proxychains] Strict chain  ...  127.0.0.1:9050  ...  whatismyip.akamai.com:80  ...  OK
96.44.XXX.XXX
```

I've censored the IP addresses, but you can see that they are completely different once proxychains is used.
Tor is not a magic bullet, and it does have some limitations. Please read their [tips](https://www.torproject.org/download/download.html.en#warning) to ensure that you're not leaking any information when using Tor. You will find that sites load slower when using Tor - this is normal because you're going through several proxies to reach your target site. Another downside is that some servers will block Tor exit nodes, or will add a speedbump along the way (Google will sometimes make you complete CAPTCHAS before you can use their service).

Next up, anonymizing email. Almost every service wants your email address before they provide the service for you. It doesn't matter if you just want a coupon for a coffee, or if you want to download beta software. For anonymous email, I like to use [GuerrillaMail](http://www.guerrillamail.com/). Using it is easy, you go to their site and you're instantly given a random temporary email account that's good for one hour, and as long as you have that browser window open. Now you can sign up for all those coupons without having to worry about giving away your real email address.

Finally, encrypted chatting. I recommend [CryptoCat](https://crypto.cat/). It's an online chat service where you create your own room and give that room name to the person you want to chat with. You both log on, and conversations between the two of you are encrypted. You can even use the service to exchange files securely between one another.

The Internet is becoming more social, and new policies are being considered that threaten our digital privacy. Anonymizing tools and services such as Tor provide a way to escape Big Brother's growing reach.
