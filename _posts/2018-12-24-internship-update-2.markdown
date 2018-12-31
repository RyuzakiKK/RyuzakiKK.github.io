---
title:  "GNOME Security Internship - Update 2"
date:   2018-12-31 16:35:00 +0100
category: GNOME
tags:
  - GNOME
---

The introduction and the first update can be found [here](/gnome/internship-preparation/) and [here](/gnome/internship-update-1/).

## Allow one keyboard even when screen is locked
Let's hypothesize that you choose to protect your PC from new USB devices when the lock screen is active.
USBGuard does its job and every USB devices plugged with a locked screen gets blocked.
The key word here is *every*.

What if your keyboard breaks? You go to your garage searching for an old working keyboard.
After 15 minutes of searching, you find it and you return to your PC.
The screen is now locked because the automatic timeout passed.
You plug the new keyboard but it doesn't work, because the screen is locked.

You could do a reboot but in this way you'll lose all your unsaved work.

In order to gracefully handle this situation we added an exception to the block rule.
Now if you plug in a keyboard we authorize it if, and only if, this is the only one currently available.

And what if an attacker disconnects my keyboard, plugs his infected device and then reconnect my keyboard?
In this scenario the attacker device will be authorized (if it advertise itself as a keyboard), but then the session would be still locked.
So he needs to reconnect your keyboard too, because he needs to wait that you return in order to unlock the PC.
But when he reconnects your keyboard it will not be authorized because it is no more the only available keyboard in the system.
Also because we don't permanently store a list of whitelisted devices.
So when you replug a device it will be treated as new, like it never saw it before.

In order to check if the plugged device is a keyboard we check if the USB class is "03:00:01" or "03:01:01", as described in the USB specs.

Are there currently some problems to this?
Unfortunately yes, because it's not everything black and white.
This method works very well until your "gaming" programmable mouse also advertise itself as a keyboard.
In this scenario we do not authorize your replacement keyboard because we see that there is already a working one.

In the end this solution definitely improves the situation, while in the future probably it could be revised and tweaked a bit more.


## Initial work regarding a "smart" always block.
This is something still in its early stage.
In a first version, if the screen is not locked and you plug a keyboard the screen will be locked.

This works good in theory but not in practice, because not only devices with physical keys are keyboards.
For example the devices for hardware 2FA (e.g. yubikey) are also keyboards, and locking the screen every time you plug one of those is not a pleasant experience.
So for a first implementation this solution is ok, but it definitely needs to be improved.

One way to do it can be by [mapping scancodes to keycodes](https://wiki.archlinux.org/index.php/Map_scancodes_to_keycodes), limiting particular devices capabilities (e.g. prevent them to use risky keys like "alt" or "ctrl").
Anyway this will require more research about what we can do and what's the best way to do it.


## How do we notice USBGuard configuration changes?
Before this part was handled by GNOME-Control-Center (g-c-c).
Meaning that it had the job of keeping in sync what we had in gsettings and the internal state of USBGuard.
But this was a problem if, for example, the user manually changed the USBGuard configuration while g-c-c was closed.
We would have noticed the change only at the next g-c-c opening.

### Refactoring!
In order to solve this a quite substantial refactoring happened.
Now we have:

- GNOME-Control-Center: part of its logic migrated to GNOME-Settings-Daemon. Now it just syncs with the gsettings schemas and doesn't talk anymore directly with USBGuard.

- gsettings-desktop-schemas: as before we store here under org.gnome.desktop.privacy the desired USB protection level.

- GNOME-Shell: previously g-s had the job to authorize new USB devices, mainly because this was the only component that was always running.
       Now g-s is just used to display an indicator icon when the USB protection is active.

- GNOME-Settings-Daemon: this is the new entry, now we have a daemon always running.
         It has two jobs, it keeps in sync the USBGuard configuration with the schemas on gsettings and also it is the one that authorizes new USB devices.

![USB protection status icon](/assets/images/usb-protection-icon.png)

## Enable and disable USB protection
From GNOME-Control-Center we give the user the ability to set the protection level to:

- "Never": meaning that you don't want any sort of protection. You want every new devices to be authorized.

- "When lockscreen is active": meaning that you want the protection only when the session is locked.

- "Always": meaning that you always want the protection.

With "never" we set USBGuard's `InsertedDevicePolicy` to `apply-policy` and we prepend to the rule file `allow id *:*`.

Good, but what if I want to manually configure USBGuard and I don't want GNOME to mess anymore with my config, how do I do it?

To handle this scenario we added a more explicit on/off switch to g-c-c.
"On" actually means "I want GNOME to handle USBGuard".
On the other hand "off" means "I want to handle USBGuard by myself".

![g-c-c switch](/assets/images/g-c-c-usb-switch.png)


## What to expect next and comments
In the next days I'll try to improve the always on protection.
Also I need to add a more robust way to check if we have a working USBGuard on the system.
I'll also do some extensive testing, trying to check the behaviour with different environments and scenarios.

Feedback is welcome!

Happy new year!