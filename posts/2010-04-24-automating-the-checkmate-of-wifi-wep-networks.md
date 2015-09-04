
The techniques used to crack WEP vary depending on whether or not it uses MAC filtering, if it uses shared key or open key authentication, if we need to perform a deauthentication, and so on. I've found that the majority of WEP networks I've audited usually don't employ MAC filtering and they use open key authentication. The series of steps I begin with goes like this.

<!--more-->

1. Scan for WEP networks, pick a target, and start capturing packets.
2. Do a fake authentication, this takes care of whether or not there are clients associated with the access point.
3. Perform a fragmentation attack so we can build an ARP packet.
4. Use packetforge-ng to create an ARP packet that we can reply.
5. Replay the ARP packet and start generating unique IVs.
6. Start cracking.

Eight out of ten times, I've found that this sequence of steps gets me the WEP key within a few minutes. However, running that sequence over and over again for every WEP protected access point gets tedious very quickly. Since it works most of the time, I decided to script the entire procedure. This allows me to crack most WEP access points and leaves me time to investigate the ones that it can't crack. You can grab the latest version from [GitHub](https://github.com/superkojiman/crack_wep). 

This was tested on BackTrack 4 Final Release. You'll need a wireless adapter that can re-inject packets - I used the [Alfa AWUS036H](http://www.data-alliance.net/-strse-73/Alfa-AWUS036H-500mW-USB/Detail.bok) which worked perfectly. I'm also going to assume you know how to use the [aircrack-ng](http://www.aircrack-ng.org/doku.php?id=simple_wep_crack&DokuWiki=419b234c6dbc46209f8197defe4f6d20) suite.

The script is easy enough to use. Set the Alfa into monitor mode and pick out a target and start capturing packets with airodump-ng. Open another terminal and run the script. It takes two parameters, the first is the target access point's MAC address, and the second is the Alfa's interface; in our case it's mon0. 

At this point the script will do everything and you just have to sit back and wait to see if it works. Once it starts replaying the ARP packets and you see the IVs increasing, run aircrack-ng on the IVS dump file and wait for it to crack the key. 

If the script fails, maybe the access point uses shared key authentication, or maybe it uses MAC address filtering. The script doesn't take those scenarios in account since I hardly run into them so I don't mind investigating those manually.
