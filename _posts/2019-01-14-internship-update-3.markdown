---
title:  "GNOME Security Internship - Update 3"
date:   2019-01-14 16:25:00 +0100
category: GNOME
tags:
  - GNOME
---

Here you can find the [introduction](/gnome/internship-preparation/), [the update 1](/gnome/internship-update-1/) and [the update 2](/gnome/internship-update-2/).

## Notification if you replug your main keyboard
As of now we allow a single keyboard even if the protection is active because we don't want to lock out the users.
But Saltarelli left a comment making me notice that an attacker would have been able to plug an hardware keylogger between the keyboard and the PC without the user noticing.

To prevent this now we display a notification if the main keyboard gets unplugged and plugged again.

![USB new keyboard notification](/assets/images/usb-new-keyboard-notification.png)


## Smart authorization with touchscreen
If your device can use the touchscreen and your physical keyboard breaks you should be able to use your device because you'll have a working on-screen keyboard.
Because of this I added an exception to the auto authorization of keyboards if touchscreen is also available.
Thanks to Marcus for this hint.


## GNOME Shell protection status
Now GNOME-Shell shows an icon in the status bar only if the protection is really active.
In order to reliably test it we check both the gsetting protection value and also if the USBGuard DBus is really up and running.

As shown in the screenshot below we are currently using a generic "external device" icon.
Before the final release we should change it, hopefully with an external help :)

![USB protection status icon](/assets/images/usb-protection-icon2.gif)


## GNOME Control Center
GNOME Control Center before showed just an "on"/"off" label next to the "forbid new USB devices" entry.
Now a more informative label has been added.
In this way we can show directly the protection level in use.

On top of that, we also check for the USBGuard DBus availability.
If it is not available we show the current protection level as "off" and we prevent the users to interact with the USB protection dialog.

![g-c-c USB current status](/assets/images/g-c-c-usb-current-state.gif)


## Limit keyboards capabilities
As I said briefly in the last update, the goal here is to prevent new keyboards from using dangerous keys (e.g. ctrl, alt ecc...).

At the beginning I tried to experiment with [scancodes](https://wiki.archlinux.org/index.php/Map_scancodes_to_keycodes).
One possible approach was to append to the hwdb an always limit rule like this:

```
evdev:name:*:dmi:bvn*:bvr*:bd*:svn*:pn*    # Matches every keyboards
    KEYBOARD_KEY_700e0=blocked             # Block left ctrl
    [...]
```

Then when we wanted to grant full access to a single keyboard we appended a device specific rule (vendor, product, version ID and input-modalias of it) mapping back every previously blocked keys.

While at first this seemed a feasible option, after a few emails and a call with Peter Hutterer and Benjamin Tissoires we decided to discard it in favour of a device grabbing with EVIOCGRAB/libevdev_grab().

For example one problem with scancodes mapping using the hwdb was that the matches were not predictable if there were multiple of them applied to a single device (first the always block and then the permission granted).

### EVIOCGRAB and Mutter
The route we are taking is to implement the key filtering directly in mutter ([my early work on it](https://gitlab.gnome.org/denittis/mutter/tree/usb_protection)).
As soon as we have a new device we take the grab on it with EVIOCGRAB, in this way we will be the only one that can see the device key events.
Then when a key gets pressed we check if it is a dangerous key.
If it is we drop it.

With this approach for example we still can get notified when a dangerous key is pressed (as opposite to the scancodes approach).
In this way we can display a notification saying that the device is limited, so nothing happened because a key has been blocked and not because the keyboard is broken.

How to handle this in the UI is still under discussion and evaluation.
Maybe we can start with showing a list of connected keyboards in GNOME Control Center.
Next to every entry in this list we could put a colored icon to show the current device status.
That is could be either limited or full access.
Then the users will be able to toggle this icon changing the specific device permission.


## What to expect next and comments
In the next days I'll try to refine the "limit keyboards" functionality and I'll try to come up with a way to expose this feature to the end users.
Also related to the smart authorization, we should also check for other input devices like keyboards on the legacy PS/2 port.

Feedback is welcome!