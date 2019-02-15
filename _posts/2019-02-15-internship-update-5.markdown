---
title:  "GNOME Security Internship - Update 5"
date:   2019-02-15 14:45:00 +0100
category: GNOME
tags:
  - GNOME
---

Here you can find the [introduction](/gnome/internship-preparation/), [the update 1](/gnome/internship-update-1/), [the update 2](/gnome/internship-update-2/), [the update 3](/gnome/internship-update-3/) and [the update 4](/gnome/internship-update-4/).

## Always on protection vs lock screen protection
This project started with a simple on/off switch in control center that entirely enabled or disabled the USB protection.
A respectively so called always on and always off.

Later on we introduced a smarter protection level that was active only when the user session was locked.

While an always on protection seemed a good idea on paper it turned out that the advantages compared to the lock screen protection were very slim.

When the screen is locked both protections have the same behaviour.
They only differentiate when the user session is unlocked.

With "always on" protection you are covered even when you leave your pc unattended and unlocked, right?
Well, not really.
The protection level is configurable from Control Center without root access (because we can use USBGuard's D-Bus with user privileges).
This means that if you leave your pc unlocked an attacker could just disable the protection and then plug his malicious device.
With this in mind, the always on protection can give us more troubles than benefits (angry users that can't plug their devices).

### So should we completely remove the always on protection? 
From GNOME Control Center probably yes.

But there are use cases where the always on protection can still be useful, like in a kiosk setup where you never expect USB devices to be plugged in and where the users can't reach Control Center.
So probably we will keep this protection level only as a gsetting option.

### How to show the lock screen protection on GNOME Control Center
Dropping the always on protection will leave us with just the lock screen one.
It means that we can simplify the dialog on Control Center.
Instead of a dropdown menu we can just show an on/off switch, similarly to what we had in the early proof of concept for this project.


## Protection enabled by default?
There is a currently ongoing [discussion](https://gitlab.gnome.org/GNOME/gnome-shell/merge_requests/369) about enabling the USB protection by default.
This is also for example what is starting to happen with ChromeOS (I'll talk about it below).

### If we enable the protection by default, can we remove it from Control Center?
Probably not.
Even if we have this option enabled for everyone, some users may still want a custom USB protection behaviour.
For example they could use a USBGuard GUI or manually edit the USBGuard configuration.
In this case we can't also interfere and mess with USBGuard configuration.

So we still want to give users an easy way to manually manage USBGuard or even disable it completely.

If this option should be exposed from Control Center, or only with a gsetting, is something that probably we still need to evaluate.


## ChromeOS update
In the [project wiki page](https://wiki.gnome.org/Internships/2018/Projects/USB-Protection) we mentioned that ChromeOS was going to integrate USBGuard support.
Now it already landed in the Canary channel and can be enabled with the flag `chrome://flags/#enable-usbguard`.
In the future probably it will be enabled by default.

To understand ChromeOS approach I checked their source code and this is a quick summary of what I found:
1. When the screen is not locked USBGuard daemon is stopped/killed.
2. When the screen gets locked USBGuard daemon is started with a custom `rules.conf` file.

What's inside `rules.conf`? I.e. what type of devices are blocked?
```
allow with-interface one-of { 03:00:01 03:01:01 } # Whitelist keyboards.
block with-interface one-of { 05:*:* 06:*:* 07:*:* 08:*:* } # physical, image, printer, storage
allow
```

This can be summed up as "allow everything except for physical, image, printer and storage USB classes".
 
In what our solution differs?
- The first main difference is that we edit USBGuard configuration to let it run even when the lock screen is off.
In this way we don't need to stop and start it after every lock screen change of state.
- The second difference is regarding the whitelist.
Currently to prevent locking out users when their main keyboard breaks we allow a new one if, and only if, it is the sole currently available.
ChromeOS instead does a wider whitelist on all keyboards.

Apart from the two differences highlighted above I think that it is good to see that they are using a similar approach to the one we have right now--i.e., block new USB devices when the session is locked.


## First UI PoC for keyboards protection
In the keyboard protection front (block dangerous keys for untrusted devices) this is the first Control Center UI concept that I realized.
Keep in mind that this is still heavily a work in progress.
This interface is just the first iteration.
 
![G-c-c USB Tab](/assets/images/g-c-c-usb-tab.gif)

Where to store if a device is authorized or not?
- One possibility is USBGuard itself.
When we grant a device full access we may store it in USBGuard's rule file.
With this approach we will be able later to access this list using the D-Bus method `listRules`.
- Another possibility is to create a custom DB (maybe a plain text file) where we will store these information.

Right now mutter (more precisely from clutter) is where we really drop the dangerous keys for untrusted devices.
So we also need to think about how to access this whitelist from there.

As a first implementation we will probably generate an udev rule with the hwdb file.
In this way we will be able to access the authorization property from libinput, without the need of a D-Bus call.


## What to expect next and comments
I'll edit the gnome-control-center MR to reflect the fact that we don't need anymore a dropdown menu.
Also hopefully before the next update post we will be a step closer to reach a production ready keyboard protection.

Feedback is welcome!