---
layout: post
title: "Two-factor authentication for the rest of us"
date: 2009-10-14 22:59:11 -0400
comments: true
categories: howto
alias: /2009/10/two-factor-authentication-for-rest-of.html
---

Encryption is a great way to protect your data. Generally you select some awesome algorithm, pick a passphrase and encrypt your data. When you want to access your data, you decrypt it using the passphrase. 

The problem lies in the passphrase. If your passphrase is weak, then it doesn't matter how strong the encryption is. A weak passphrase that can be easily guessed or brute forced will only give you a false sense of security. If you've got a keylogger installed on your system without you knowing, then your passphrase can get recorded as you're typing it. 

To mitigate this risk, we can use two-factor authentication. <!--more--> This requires that you present two valid factors to authenticate: something you know, and something you have. An example would be authenticating to an ATM machine with your PIN (something you know) and your ATM card (something you have). [RSA SecurID](http://www.rsa.com/node.aspx?id=1156) provides an enterprise solution for this. However this is expensive and probably overkill for most home users. 

Enter [TrueCrypt](http://www.truecrypt.org). TrueCrypt is a "free open-source disk encryption software for Windows Vista/XP, Mac OS X, and Linux". TrueCrypt allows you to create an encrypted volume to store your sensitive files in. To decrypt it you need a passphrase and/or a keyfile. They keyfile is what makes it possible for us to use two-factor authentication with TrueCrypt. The keyfile can be any file on your system such as an MP3 or a JPEG. So here's the idea:

* Create an encrypted volume and choose to secure it with a passphrase and a keyfile.

* Store the keyfile in a USB flash drive that you can easily carry around.

* When you're ready to decrypt your data, you need to plug in the USB flash drive (something you have) and enter the password (something you know) to decrypt the volume.

At this point if you've used TrueCrypt before, you probably already know how to go about doing this. For everyone else follow the TrueCrypt [tutorial](http://www.truecrypt.org/docs/?s=tutorial), but when you get to the section on providing a passphrase, do the following:

*(By the way, I did this on a Mac, but the steps should be similar on Linux and Windows)*


* Insert your USB flash drive. Let's assume for this example that it shows as MyUSBDrive.

* You should be at the sreen where you're asked for a Volume Password. Type in a password and then check Use keyfiles and click the Keyfiles button.

* The Select Keyfiles window will pop up. Click on Generate Random Keyfile.</li>

* Another window will pop up showing the Random Pool. Move your mouse around that window for a bit and then click Generate and Save keyfile. We'll call it mykey

* Now you need to specify the location. Head over to your USB flash drive and save it there. You'll get a message saying the keyfile was successfully created. You can close the Random Pool window now.

* Back at the Select Keyfiles window, click on Add Files and navigate to the newly created keyfile that you created and click OK.</li>

* You'll be back to the Volume Password window now.  So click Next and continue the rest of the tutorial.

Now whenever you mount the encrypted volume you need to plug in your USB flash drive with the keyfile and enter your passphrase. 

One problem I have with TrueCrypt is that it's a bit of a hassle to mount the encrypted volume. You have to launch TrueCrypt and click through several buttons before you can even mount it. This can be easily automated with the <a href="http://www.truecrypt.org/docs/command-line-usage">command line options</a> so that it selects the correct keyfile from your USB flash drive and prompts you for a passphrase. Wrap that in a script that you can double-click and you're good to go. I'll leave that as an exercise to the reader.  
