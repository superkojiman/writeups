
If you need to do a brute force attack against a particular service, you'll need a couple of things. A good wordlist containing possible passwords, and a list of user names to try. It's easy to get a password list on the Internet, but user lists often have to be customized for the target. You'll need to do some research to find email addresses and employee names. Once you do have a list of names however, you'll need to guess what the format of the login ID is for that user. John Doe could be johndoe, or john.doe, or jdoe, and so on. 

<!--more-->

Since having a proper user name list is just as important as having a good password list for a brute force attack, I've created a short script that will create a list of possible login IDs based on a person's first and last name.

First gather the names of people you've found who might have a login account for the service you're targetting. Each name should be on a line of it's own:

```
Cloud Strife
Brian O'Connor
Sonic The Hedgehog
```

Now here's the script that will create the possible login IDs:

{% gist 11076951 %}

Run the script by passing the file containing the first name and last name and you'll get an output that looks like this:

```
cloudstrife
strifecloud
cloud.strife
strife.cloud
cstrife
scloud
c.strife
s.cloud
brianoconnor
oconnorbrian
brian.oconnor
oconnor.brian
boconnor
obrian
b.oconnor
o.brian
sonichedgehog
hedgehogsonic
sonic.hedgehog
hedgehog.sonic
shedgehog
hsonic
s.hedgehog
h.sonic
```

Now you have a user name list that can be passed as input to cracking tools like hydra, medusa, ncrack, and Metasploit. Using a good user name list is just as important as having a good password list. If a user's password is in your password list but your user name list doesn't contain the proper format of the user name, then you're not going anywhere fast. The script is easily customizable, so if you think up of any other possible formats, feel free to add it in. 
