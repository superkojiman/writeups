
Twitter has two forms of authentication that it exposes to developers. The first is
[OAuth](http://oauth.net/) and the second
is basic authentication. 

The OAuth website describes OAuth as "... a standard way for developers to offer their services via an
API without forcing their users to expose their passwords (and other credentials)."

<!--more-->

Many popular Twitter applications use OAuth as means to authenticate the user with their Twitter account.
Although OAuth isn't without
[issues](http://blog.oauth.net/2009/04/22/acknowledgement-of-the-oauth-security-issue/), it's a lot better than the alternative: basic authentication.

Basic authentication has been around for a while. An HTTP client that needs to authenticate to a
webserver sends the username and password encoded in Base64. When used without SSL, basic authentication
provides no security.

Basic authentication has been around for a while. An HTTP client that needs to authenticate to a
webserver sends the username and password encoded in Base64. When used without SSL, basic authentication
provides no security.

Let's take a closer look at why this is a problem. Here's what a GET request sent by a Twitter
application using basic authentication looks like:

```
GET /statuses/friends.json HTTP/1.1
Accept-Encoding: identity
Host: twitter.com
Connection: close
Authorization: Basic TWlzdGVySm9obkRvZTpTZWNyZXRQYXNzd29yZA==
```

The last line is what we're interested in. The string TWlzdGVySm9obkRvZTpTZWNyZXRQYXNzd29yZA== is the
username and password encoded in Base64. The way this is created is simple. The username is concatenated
with a colon, and then concatenated with the password. Assuming the username is MrJohnDoe and the
password is SecretPassword, we would get MisterJohnDoe:SecretPassword

The concatenated username and password is then encoded in Base64. This is easily done with the openssl command, or
using an online resource (Google: Base64 decoder encoder). Let's go with openssl:

```
printf MisterJohnDoe:SecretPassword | openssl enc -base64
```

This returns TWlzdGVySm9obkRvZTpTZWNyZXRQYXNzd29yZA== which is the same string that's sent to the webserver. To
decode it:

```
printf TWlzdGVySm9obkRvZTpTZWNyZXRQYXNzd29yZA== | openssl enc -d -base64
```

This returns MisterJohnDoe:SecretPassword.

Ok, so it's trivial to encode and decode the credentials. Now, if you can find someone using a wireless network
with a Twitter application that uses basic authentication, then you can easily capture their Base64 encoded
credentials.

The procedure is simple. Capture the data using a wireless sniffer like Kismac and load the the packet capture into
Wireshark. Search for the basic authentication request and Base64 encoded credentials. Wireshark will automatically
decode it for you, or you can decode it yourself. Mission accomplished.

Fortunately, there are ways to prevent this from happening to you. One suggestion is to switch to a Twitter
application that uses OAuth instead, or a Twitter application that uses SSL and basic authentication. A quick check
on Google or the application's website will tell you if it supports OAuth. Or you could just use the Twitter.com
website to post your tweets.

So, what are *you* doing? 
