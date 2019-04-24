---
title:  "After GNOME Security Internship - Update 7"
date:   2019-04-24 17:00:00 +0100
category: GNOME
tags:
  - GNOME
---

Here you can find the [introduction](/gnome/internship-preparation/), [the update 1](/gnome/internship-update-1/), [the update 2](/gnome/internship-update-2/), [the update 3](/gnome/internship-update-3/), [the update 4](/gnome/internship-update-4/), [the update 5](/gnome/internship-update-5/) and [the update 6](/gnome/internship-update-6/).


## Part 1, protection from unwanted new USB devices

I received a few code reviews in the GNOME Settings Daemon MR that I'll try to address in the next days.

Also I'm going to widen the requirements for allowing keyboards when the screen is locked.
Right now if the lock screen is active we authorize a keyboard only if it is the only available keyboard in the system.
It was a good idea in theory but not that much in practice.

For example let's assume that you use an hardware USB switch hub between your desktop and your laptop with a mouse and a keyboard attached.
If you have a "gaming" mouse with extra keys it is not only a mouse but also a keyboard.
That means that when you want to switch from your laptop the the desktop, the mouse and the keyboard will be connected nearly simultaneously and if the mouse goes first the real keyboard will not the authorized.
So you'll be locked out from your system.

The gaming mouse is also only an example. If you have a yubikey shared in this USB hub there will be the same problem explained above.

For this reason in the next days I'll edit the current implementation so that every USB keyboards will be authorized even if the lock screen is active.
However we will still show a notification to explain that we authorized a new keyboard while the screen was locked.

> <cite> Security at the expense of usability comes at the expense of security.</cite>
>
> -AviD's Rule of Usability

## Part 2, limit untrusted USB keyboards

At the beginning of this month I started to work for Collabora and they allowed me to continue this GNOME project in my R&D time!

So thanks to that I made some interesting progress:

- I added the required new GSetting to enable/disable the keyboard protection in gsettings-desktop-schemas

- I experimented with the kernel key masking [EVIOCSMASK](https://gitlab.gnome.org/denittis/mutter/commits/usb_protection_EVIO).
It worked very well with just a couple of issues still to address.
Anyway after these tests I came to the conclusion that maybe the key filtering in the userspace is a better solution for this usecase.
For example in the future it will be easier to add extra functionalities like showing a system notification when a not authorized keyboard sends a dangerous key.

- In Mutter previously when we needed to check if a key was in the dangerous keys list we used a binary search.
But the dangerous keys list is static so I removed the binary search in favor of using an hashset.
Even if the performance gain is negligible, we can now perform a search in O(1) instead of O(log n).

- Last time I did a benchmark it was when we used a device hash with name, vendor and product id instead of using the udev property as we do now.
So I redid it and I'm glad to see that in the worst case scenario the delay of a key press halved.

- Instead of using hwdb we store the device authorization in a local db and we use an udev helper to import the rules from the db.
In this way the udev rule will always be static.

- I completed the integration with GNOME Control Center.
Now there is a working lock/unlock button to gain admin privileges via polkit.
When the user changes the authorization level of a device it gets stored in a local db and we trigger an udev reload echoing "change" into the "uevent" sysfs attribute of the device.
This new panel is also hidden if there isn't a wayland session (the mutter key filtering works only on wayland).

- I opened the necessary MRs upstream to gather feedback from the community.
[GNOME Control Center MR](https://gitlab.gnome.org/GNOME/gnome-control-center/merge_requests/462), [Mutter MR](https://gitlab.gnome.org/GNOME/mutter/merge_requests/550), [GSettings Desktop Schemas MR](https://gitlab.gnome.org/GNOME/gsettings-desktop-schemas/merge_requests/22).

<video controls> <source src="/assets/images/keyboard-protection.webm" type="video/webm" /> </video>


### Benchmark v2

Now that we simply check if a device has the udev property `GNOME_AUTHORIZED` I did another benchmark:

- Mutter stock: 1-2 usec to compute the pressed key.

- With keyboard security off: 1-2 usec. It’s just an extra `if` check.

- With keyboard security on: 1-2 usec. An extra `if` and a search in the hashset of "dangerous" keys.

- Keyboard security on and you press a "dangerous" key: 4-6 usec. In this case we also need to check the udev properties of the device.

Compared to last time, the worst case scenario execution time dropped from 7-12 usec to 4-6 usec.

