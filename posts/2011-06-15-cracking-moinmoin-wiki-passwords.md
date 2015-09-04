---
layout: post
title: "Cracking MoinMoin Wiki Passwords"
date: 2011-06-15 18:10:26 -0400
comments: true
categories: howto hacking
alias: /2011/06/cracking-moinmoin-wiki-passwords.html
---

I wanted to audit the security of a server running the MoinMoin Wiki Engine version 1.9.2 and needed to see if I could crack the passwords on the site. Each user's information is stored in a file located in the site's data/user directory, for example: 1308083750.39.64129. This is a plaintext file which contains key-value pairs. There are two keys that we're interested in: enc_password and name

<!--more-->

```
{SSHA}f5m3h686GtiwemkvPeRIMs1Kx6yWSx89iBrSvpxSFWAS9qB1kIWXzQ==
```

If we want to crack this password, we first need to know how it's encrypted. This information can be found in the encodePassword function in the user.py file:

``` python
def encodePassword(pwd, salt=None):
    """ Encode a cleartext password

    @param pwd: the cleartext password, (unicode)
    @param salt: the salt for the password (string)
    @rtype: string
    @return: the password in apache htpasswd compatible SHA-encoding,
        or None
    """
    pwd = pwd.encode('utf-8')

    if salt is None:
        salt = random_string(20)
    assert isinstance(salt, str)
    hash = hash_new('sha1', pwd)
    hash.update(salt)

    return '{SSHA}' + base64.encodestring(hash.digest() + salt).rstrip()
```

Here's what the function does:

1. Create a random salt of 20 characters
1. Hash the password using SHA-1
1. Update the hash with the salt
1. Encode the hashed password and the salt with Base64

The result is assigned as the value to the enc_password key. Now that we know how to the password is encrypted, we can write a script that will perform a dictionary attack against the encrypted password:

{% gist 11073420 %}

The script takes two arguments. The first is a colon delimited username and password. This can be created with a bit of command-line-fu from the data/user directory. It should look like this:

```
JohneDoe:/Ua/4WcsyRkxE9oeebb5XKkYy0yb7Ii/n3NHfzEhZOK88RC9rR58EQ==
LucyWilliams:9J/BpH4pNWYYu+y4dC5NF6xSdhEBtVQ7Z4DOO0cM0MuaAMvfplikUA==
KarlWalters:NlQO/svt1+wEhBdiHK9Ep6WzbGPahlICUvYgt32K4OvzvJeDYyPuTQ==
```

The second argument is a wordlist. Google dictionary.txt and you should find something to get you started. Once you have both files ready, pass them both to the script and you'll be notified if the passwords are found in the wordlist:

```
$ ./moincrack.py users.txt passwords.txt 
trying to crack password for JohnDoe
 * password for JohnDoe: lion9
trying to crack password for LucyWilliams
 * password for LucyWilliams: chickensoup
trying to crack password for KarlWalters
```

Older or newer versions of MoinMoin might encrypt the passwords differently, but you should be able to modify the script accordingly to suit your needs.
