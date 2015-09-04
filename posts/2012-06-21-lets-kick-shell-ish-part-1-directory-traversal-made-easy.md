---
layout: post
title: "Let's kick shell-ish, part 1: Directory traversal made easy"
date: 2012-06-21 18:10:26 -0400
comments: true
categories: hacking coding
alias: /2012/06/lets-kick-shell-ish-part-1-directory.html
---

Web applications that are vulnerable to directory traversals offer a small window into viewing the contents of a target server. In a way, you've semi-penetrated the system, albeit with minimal privileges, mostly just reading files. However, that's not necessarily a bad thing. Being able to read /etc/passwd for instance will give you an idea of what user accounts are on the system, thereby aiding in a brute force attack. If you can read the contents of C:\Windows\repair\sam and C:\Windows\repair\system, you can download those files and start cracking Windows passwords.

<!--more-->

Let's have a quick look at a web application that's vulnerable to a directory traversal attack. Here we've found a target that's running a vulnerable version of WebcamXP5:

->![](/images/2012-06-21/01.png)<-

Looking through Exploit-DB, we find the following page: [http://www.exploit-db.com/exploits/18510/](http://www.exploit-db.com/exploits/18510/), which details a directory traversal vulnerability in WebcamXP5. Let's attempt to read C:\boot.ini:

->![](/images/2012-06-21/02.png)<-

Hey it works! Now if we want to read another file, we need to update the URL in the browser bar. As you keep going through the list of files you want to read, you'll find that typing on the browser bar and editing the URL is slow and cumbersome. To get around this, I decided to write a script that would provide a sort of shell-ish interface to make things a lot easier and quicker. I call this directory traversal shell, dirtshell. Let's have a look:

->![](/images/2012-06-21/03.png)<-

Here we've specified the prefix (```\..\..\..``` and so on) as an argument to the program dirtshell. Once the shell starts, we can just type in the path and file that we want to read. In this case, we've read the contents of C:\boot.ini and C:\Windows\system.ini. In a way, it looks like we've got a shell in the server itself.
Ok so that's great, but the process of typing each file and hitting Enter to view the result still takes a little time. dirtshell can take a file with a list of files that we want to read. For example, let's create a file called check.txt with the following:

```
\\boot.ini
\\windows\\win.ini
\\windows\\system.ini
```

We can tell dirtshell to read that file and just print the output of each file specified if it exists. No interaction with the user required:

->![](/images/2012-06-21/04.png)<-

In this way, we can sort of use it as a fuzzer, or if we have a list of files that we're interested in, we can just put those in the file and have dirtshell automatically read them for us. You can download the latest version of the script from [GitHub](https://github.com/superkojiman/dirtshell). 

dirtshell has three options that can be specied. The -u specifies the URL that you're targetting, so it's mandatory. The -p option as we've already seen, specifies the prefix such as "```../../../```" or "```\..\..\..```". The -s option specifies the suffix, for instance "" when dealing with php files that need to be null terminated.
