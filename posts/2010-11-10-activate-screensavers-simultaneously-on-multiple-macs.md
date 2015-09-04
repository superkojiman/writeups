---
layout: post
title: "Activate screensavers simultaneously on multiple Macs"
date: 2010-11-10 18:10:26 -0400
comments: true
categories: coding
alias: /2010/11/activate-screensavers-simultaneously-on.html
---

My work setup in the lab consists of two Mac Minis, one Mac Pro, and my Macbook Pro. I do all my typing on my Macbook and use [teleport](http://www.abyssoft.com/software/teleport/) to remotely control the other computers. Whenever I leave my desk, I make it a habit to lock all my computers using the screensaver. It has become a bit of a pain to do this manually, so I came up with a way to lock all my computers with a single command from my Macbook. 

<!--more-->

The idea is to use ssh to remotely run a command that will activate the screensaver on the target computer. In order for this to work properly, SSH keys should be setup between the computer sending the lock request (my Macbook) and the targets (Mac Minis and Mac Pro) so that no password input is required. Read up on ssh-keygen and ssh-agent for details on how to do this.

Here's the script:

```bash
#!/bin/bash
function lock_request {
    user=$1
    target=$2
 
    # lock the screen only if the target is alive.
    ping -c1 $target 2>&1 > /dev/null
    if [[ $? -eq 0 ]]; then
        ssh ${user}@${target} open /System/Library/Frameworks/ScreenSaver.framework/Versions/A/Resources/ScreenSaverEngine.app
    fi
}
 
# lock remote targets
lock_request johndoe 192.168.123.181 &
lock_request johndoe 192.168.123.168 &
lock_request johndoe 192.168.123.146 &
 
# lock local screen
open -a ScreenSaverEngine
```

The lock_request function expects two parameters, the login name and the IP address of the target. The first thing the function does is to ping the target to see if it's alive. If it is, it connects to the target using ssh and executes the open command on ScreenSaverEngine.app which is what launches the screensaver. Once all targets are locked, it locks the local computer as well. 

Feel free to customize and improve on it. 
