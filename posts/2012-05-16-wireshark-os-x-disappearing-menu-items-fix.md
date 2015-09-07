---
layout: post
title: "Wireshark OS X: Disappearing menu items fix"
date: 2012-05-16 18:10:26 -0400
comments: true
categories: howto
alias: /2012/05/wireshark-os-x-disappearing-menu-items.html
---

Wireshark on OS X runs on top of X11. As most people who've used X11 applications on OS X are aware, they look ugly, and don't match the theme on OS X. In an effort to prettify Wireshark, the developers have included a default theme to go with it: Clearlooks-Quicksilver-OSX. At first, this looks nice, up until you actually start using any of the menu items on Wireshark. The text just disappears. White text on white background. Have a look:

<!--more-->

![](/images/2012-05-16/01.png)

This gets annoying quickly. Here's a quick fix:

```
cd /Applications/Wireshark.app/Contents/Resources/themes/Clearlooks-Quicksilver-OSX/gtk-2.0
sudo mv gtkrc gtkrc.orig
sudo echo 'gtk-font-name="Lucida Grande 12"' > gtkrc
```

Restart Wireshark and you now have readable menu items:

![](/images/2012-05-16/02.png)

You might have noticed that the white theme itself has been replaced by a grey one since we've blown away the Clearlooks Quicksilver theme. Oh well. Maybe there's a better way to do it, but this works and makes using Wireshark on OS X a lot less annoying. 
