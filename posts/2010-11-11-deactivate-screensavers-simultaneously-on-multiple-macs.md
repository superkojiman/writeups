
In the [last](/2010/11/10/activate-screensavers-simultaneously-on-multiple-macs)  post I discussed a method for activating screensavers remotely on multiple Macs. Turns out that it's just as much of a hassle to deactivate them, particularly if the screensaver is just meant to hide the desktop and not lock it. If you need to unlock the screen then this script will do you no good.

<!--more-->

Here's a function that can be added into the lock screen script. It uses AppleScript to simulate pressing the Enter key which deactivates the screensaver:

```bash
function wakeup {
    user=$1
    target=$2
 
    ping -c1 $target 2>&1 > /dev/null
    if [[ $? -eq 0 ]]; then
        ssh ${user}@${target} arch -i386 osascript << EOF
        tell application "System Events"
            activate
            key code 36
        end tell
EOF
    fi
}
```

Problem solved. 
