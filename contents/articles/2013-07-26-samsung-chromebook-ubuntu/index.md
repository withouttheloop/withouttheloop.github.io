---
title: Ubuntu Linux on the Samsung Chromebook
author: Liam McLennan
date: 2013-07-26 20:32
template: article.jade
---

Are you looking for a cheap, poorly made, underpowered ultrabook? If yes, then the Samsung Chromebook is for you!

It may be a bit slow, and it may be rough, but I love this thing. It is amazingly fast considering it has the processor from a 10" tablet. The 1366 x 768 display has a narrow viewing angle but is otherwise quite nice. The chromebook weighs 1.1kg and has a 6 hour battery life.

<img src="/articles/2013-07-26-samsung-chromebook-ubuntu/chromebook.jpg" alt="Samsung Chromebook"/>

The chromebook I like, and ChromeOS even grew on me somewhat, but I bought my chromebook to run Linux. 

Converting the Samsung Chromebook to Ubuntu
----------------------------

First I tried Crouton, which is a side-by-side Ubuntu with ChromeOS, but the Ubuntu experience was third rate. I had better luck with [ChrUbuntu](http://chromeos-cr48.blogspot.com.au/2013/05/chrubuntu-one-script-to-rule-them-all_31.html), which is an Ubuntu installer for Chromebooks. The default installation works well but I was left with some minor issues. 

### Fixing the Trackpad

After installing ChrUbuntu the trackpad will be rubbish. It is not hard to [find a fix on the internet](http://craigerrington.com/blog/fixing-touchpad-issues-on-arm-chromebook-chrubuntu/). After running that fix my trackpad worked great. In fact, it is one of the best trackpads I have used. 

### Fixing Bash

The terminal was driving me nuts. The up arrow didn't retrieve history and tab completion didn't work. So I posted a question on the usually helpful [askubuntu](http://craigerrington.com/blog/fixing-touchpad-issues-on-arm-chromebook-chrubuntu/) where unfortunately my question was closed, because apparently ChrUbuntu is not an Ubuntu. 

Could have fooled me.

<img src="/articles/2013-07-26-samsung-chromebook-ubuntu/screen.png" alt="Not Ubuntu"/>

Anyway I [found the solution myself](http://askubuntu.com/questions/165143/install-history-command-package). The problem is that ChrUbuntu doesn't use bash for the terminal. So install bash:

    sudo apt-get install bash

then set bash as the default terminal:

    chsh -s /bin/bash

Enjoy.
