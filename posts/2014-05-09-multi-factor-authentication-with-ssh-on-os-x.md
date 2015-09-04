---
layout: post
title: "Multi-Factor Authentication with SSH on OS X"
date: 2014-05-09 19:44:52 -0400
comments: true
categories: howto
---

This is a quick guide on how to setup multi-factor authentication with SSH using Google Authenticator. The goal is to require three items from the user in order to complete the authentication: SSH authentication keys, the user's password, and a one-time password using Google Authenticator. 

<!--more-->

This guide was tested with OS X 10.9 (Mavericks). Your mileage may vary if you try it on older releases. 

You'll need to have OpenSSH 6.2 installed. 

```
kermit $ ssh -V
OpenSSH_6.2p2, OSSLShim 0.9.8r 8 Dec 2011
```

This requirement is necessary for multiple authentication methods using the AuthenticationMethods option. This allows us to use Google Authenticator with SSH authentication keys. 

Next you'll need to have Xcode installed, or at least the command line utilities. On Mavericks this can be done using the following command: 

```
xcode-select --install 
```

Xcode is needed to build the Google Authenticator PAM module, which you should download from [https://code.google.com/p/google-authenticator/](https://code.google.com/p/google-authenticator/). 

The following commands will unpack and build the PAM module, and the google-authenticator program:

```
tar xvjf libpam-google-authenticator-1.0-source.tar.bz2
cd libpam-google-authenticator-1.0
make
```

You'll get a couple of warnings but it should build without any errors. Copy the generated pam_google_authenticator.so to /usr/lib/pam

```
sudo cp pam_google_authenticator.so /usr/lib/pam
```

Next, add an entry for it in /etc/pam.d/sshd:

```
sudo echo "auth required pam_google_authenticator.so" >> /etc/pam.d/sshd
```

Make sure the following options in /etc/sshd_config are set as follows, and if they're not in there, just add them in: 

```
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```

Restart sshd so the new configuration takes effect: 

```
launchctl unload /System/Library/LaunchDaemons/ssh.plist
launchctl load /System/Library/LaunchDaemons/ssh.plist
```

Each user that wants to use Google Authenticator with SSH needs to run the google-authenticator program, which is in the libpam-google-authenticator-1.0 directory. google-authenticator will setup the secret key and scratch codes for the user. You'll be asked a series of questions, answer yes to the first one: 

```
$ ./google-authenticator

Do you want authentication tokens to be time-based (y/n) y
https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/superkojiman@kermit%3Fsecret%3D2XCV3D3MITWANYSJ
Your new secret key is: 2XCV3D3MITWANYSJ
Your verification code is 395242
Your emergency scratch codes are:
  25217807
  58552321
  32554183
  97928439
  13891386
```

This first part is important. A URL and a secret key are generated, and these need to be input into the Google Authenticator application on your mobile device. 

The URL will display a QR code. Scanning this with Google Authenticator is the preferred way to quickly start generating the one-time passwords. If you don't want to scan the QR code, you can manually enter the secret key. 

The scratch codes can be used in the event that you are without the Google Authenticator application and need to login. We can ignore the verification code since we won't be needing it. 

You can write this information down, but you'll also find the secret key and scratch codes in your ~/.google_authenticator file at the end of the setup. 

Answer the next few questions as follows: 

```
Do you want me to update your "/Users/superkojiman/.google_authenticator" file (y/n) y

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default
size of 1:30min to about 4min. Do you want to do so (y/n) n

If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y
```

At this point you're done with the server setup. If you haven't already, scan the QR code using Google Authenticator on your mobile device so it can start generating one-time passwords. Let's test and see if the setup worked. 

I setup sshd and Google Authenticator on a machine called kermit and will attempt to SSH to it using a machine called gonzo. First let's see what happens if we attempt to SSH in without specifying an SSH private key. By default ssh looks for ~/.ssh/id_rsa but I intentionally renamed it to ~/.ssh/kermit_id_rsa so ssh wouldn't be able to find it, thereby forcing it to use password authentication only: 

```
gonzo $ ssh superkojiman@192.168.81.201
Permission denied (publickey).
```

That looks good. The conenction is terminated when the user is missing the SSH private key. Now let's try it again, this time specifying the SSH private key using -i:

```
gonzo $ ssh -i ~/.ssh/kermit_id_rsa superkojiman@192.168.81.201
Authenticated with partial success.
Password:
Verification code:
Last login: Sat May 10 01:21:38 2014 from 192.168.81.181

kermit $
```

This time it allowed the connection and prompted for a password and a verification code. To complete the authentication, I entered my account password for kermit, and the one-time password generated by Google Authenticator. 

If you enter the wrong password or the wrong verification code, you'll be prompted to start over. Note that if you've said yes to enabling rate-limiting when asked by google-authenticator, you'll need to wait 30 seconds after 3 failed attempts before you'll be prompted for the verification code again. 

That's all there is to it, you now have multi-factor authentication on SSH using the Google Authenticator PAM module. I should note that you're not limited to using Google Authenticator to generate one-time passwords. In fact, I use an alternative called [Authy](https://www.authy.com) which does the same thing but has more features. 
