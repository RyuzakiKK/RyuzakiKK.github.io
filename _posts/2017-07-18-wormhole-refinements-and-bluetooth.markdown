---
title:  "Keysign: Magic Wormhole refinements and initial Bluetooth implementation"
date:   2017-07-18 17:45:00 +0100
category: GNOME
tags:
  - GNOME
---

## User experience improvements

#### Re designing how to switch between Avahi and Wormhole
----
Previously Avahi was the default method and after a user had chosen the key to send he was able to press the Internet switch button to enable Magic Wormhole.

To allow users to choose in advance I replaced the Internet switch with an Internet toggle button in the top bar

Before

![wormhole button](/assets/images/avahi-worm-after.gif)

After

![wormhole toggle](/assets/images/avahi-worm-toggle.png)

----
#### Internet switch now means Internet+LAN
----
It may happen that the sender selects the Internet toggle because he has the abity to use it, but the receiver can only connect to the LAN.
To cover this use case, and also because a local Avahi server is very lightweight, now even with the Internet toggle enabled, the Avahi server will always be enabled

----
#### Separate information about slow and no connection
----
With the latest stable version of Magic Wormhole we are now able to immediately get the event of no Internet connection.
Thanks to this now the infobar is more precise regarding the error that occurred.

![slow connection](/assets/images/infobar2-slow.png)

![no connection](/assets/images/infobar2-noconn.png)


## Initial Bluetooth implementation
---
The development of Bluetooth started and is in an early early stage, but the discovery and the transfer is already working :)

## Work in progress under the hood
---
This week I tried to start a refactoring of the womhole part for harmonize it to the rest of the code.
As a work in progress I'm evaluating the benefit of switching from the callback mode to inline callbacks of twisted.
