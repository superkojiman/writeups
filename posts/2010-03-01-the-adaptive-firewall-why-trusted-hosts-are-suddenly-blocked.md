
Leopard Server and Snow Leopard Server have feature called an adaptive firewall (af). When the system detects
several failed login attempts from the same IP address, the af kicks in and blocks the offending IP address for 15
minutes.

<!--more-->

So why is this bad? The af is poorly documented, is not configurable through Server Admin within the firewall
service, and will most likely take an admin by surprise when hosts that are supposed to be trusted are suddenly
getting blocked. 

Querying Google for information on this yields a few relevant results. However one gem in particular stood out. A
[presentation](http://www.powerofmac.com/IT825-firewall.pdf) by Sara Porter and Mike Sebastian sheds a bit more insight into this feature.

Here's what you need to know:

1. The af is really /usr/libexec/afctl  and it comes with a man page afctl(8)  which I
suggest reading.

2. afctl  is executed by emond . The configuration file that emond  uses to
control the af can be found in /etc/emond.d/rules/AdaptiveFirewall.plist  If you go through this file
you'll see some interesting variables that you can tweak such as the default time to live (ttl) for IP addresses in
the blacklist and logging configuration.

3. afctl has its own configuration file located in /etc/af.plist There's also a man page for this: af.plist(5). One
interesting thing to note about af.plist is the variable start_behavior. According to the man page this is set to
disable by default which prevents af from starting. On a fresh install of Snow Leopard Server, this is set to enable. Note that the AdaptiveFirewall.plist overrides the ttl set in the af.plist because it passes the ttl as a
parameter to afctl using the -t flag.

4. The black/white lists are located in /var/db/af  The format for both files are different. For the
whitelist, each line contains one IP address. For the blacklist, each line contains three columns: the IP address,
the time and date to remove the IP address from the blacklist, and the ipfw  rule number. Do not edit
these files directly. Use afctl  instead to add/remove entries to them.

5. On Snow Leopard Server, 10 failed SSH attempts will automatically trigger the af and block the offending IP. If
you decide to test this, keep in mind that each SSH attempt that uses a password to authenticate actually prompts
you for a password three times. This means that to get the af to kick in, you need to enter the wrong password 30 times.  
6. Ok so what happens after your IP address is blacklisted? You can remove it from the blacklist using afctl -r, 
or you can wait for the ttl to expire and the IP address will be removed automatically. In either case, if another
failed authentication attempt occurs right after the IP address has been removed from the blacklist, the IP address
will be blacklisted again immediately. The only way to break the cycle is to have a successful authentication after
the IP address has been removed from the blacklist.

Ok, so what to do? Three possible solutions:

1. Turn off the firewall from Server Admin. Bad idea for obvious reasons. 

2. Disable af by editing AdaptiveFirewall.plist and setting all instances of Active to 0

3. Whitelist your network. Eg: /usr/libexec/afctl -w 192.168.0.0/24

My suggestion is to either disable it or whitelist your entire network and run a third party IDS like
[OSSEC](http://www.ossec.net/). OSSEC
is capable of performing various actions based on different criteria (eg: blacklist an IP address and fire off an
email to an admin after 3 failed authentication attempts).

Perhaps in a future OS X release Apple will integrate the af configuration options into Server Admin and have better documentation available for it. Until then I recommend turning it off and running a real IDS in its place. 

