---
title: JoinLater for Dummies
date: 2023-10-16
draft: false
---

Note from future me: this article is now pretty irrelevant because [I helped write some code and documentation](https://github.com/Xeyler/joinlater/pull/2/files) to have JoinLater automatically configure itself with NetworkManager! Pretty neat.

## Intro

SecureW2's JoinNow is used across many college campuses to enroll and configure personal smartphones, tablets, and laptops to secure "eduroam" wireless networks. Unfortunately, their support for Linux is limited to devices running GNOME and systemd, which is problematic for many FOSS enthusiasts who love to overly rice their obscure distro of choice.

That's where [JoinLater](https://github.com/Xeyler/joinlater) comes in, an alternative to SecureW2's JoinNow. JoinLater gives Linux users the freedom to use whatever desktop environment or init process they choose. The script at Brigham's Github is specific to USU's network, but it could be ported to other campuses if someone wants to take up that job.

The process of installing can be confusing, though. So here's a breakdown for dummies like me, mainly so I don't forget how to do this in the future.

## Debian-based distros

The following commands will get you started:

```
# Dependencies
sudo apt update
sudo apt install python3-pip python3-venv -y
mkdir ~/.joinlater && cd ~/.joinlater

# Make a virtual environment to avoid conflicting with apt
python3 -m venv joinlater
source joinlater/bin/activate

# Installation
pip install https://github.com/Xeyler/joinlater/releases/download/v0.2.1/joinlater-0.2.1.tar.gz
joinlater
```

Follow the instructions printed to the terminal. After your certs are generated, you should be set as long as you're using NetworkManager. Hooray!

## Arch-based distros

If you're using Arch, you probably don't need your hand held.

## Conclusion

For all three people that ever read this article, I hope you found it useful. Thanks!
