---
title:  "GNOME Security Internship - The Beginning"
date:   2018-11-28 14:45:00 +0100
category: GNOME
tags:
  - GNOME
---

## Introduction
For this very first round of [GNOME Internships](https://wiki.gnome.org/Internships) I'll work in the [USB Protection](https://wiki.gnome.org/Internships/2018/Projects/USB-Protection) project.

The attack surface of USB is quite large and while disabling USB altogether solves the problem, it creates other.
As do existing protection mechanisms. They suffer from poor usability and missing integration into the operating system.

This is why here we are trying a different approach to defend against rogue USB devices for a GNOME-based operating system.

USB is arguably to most exposed interface of a user's machine.
It allows an attacker to interact with pretty much any driver, many of which are of questionable quality.

In order to protect the users, while keeping the entire system easy to use, we try to be smart about when to allow new USB devices.
First, we try to detect when the user is present, arguing that if the user is not then new USB devices should not work.
But even this apparently simple case can hide some complications, because for example you might very well want to attach a new keyboard in case yours breaks.
Keyboards, however, pose another risk as several attacks have shown.

This project will be handled with an incremental build model, decomposing the whole problem in parts.
In this way we'll be able to start tackling the problem from the easier cases and gradually increase the proposed solution complexity.


## USBGuard
In this "preparation" month I started to look into USBGuard, because it is very likely that this will be the component we will use within GNOME in order to protect the USB ports.

The configuration of USBGuard consists of two files: `usbguard-daemon.conf` and `rules.conf`.
The former holds a few variables like `InsertedDevicePolicy` and `ImplicitPolicyTarget`, while the latter is a file where it's possible to whitelist/blacklist single, or groups of, USB devices.

When you plug in a new USB device in your system, USBGuard performs a few sequential checks in order to decide if a device should be enabled:

1. From the `usbguard-daemon.conf` the value of `InsertedDevicePolicy` will be checked:

   * If it is `apply-policy` it will go to the step 2.

   * If it is `block` or `reject` the device will be, respectively, deauthorized or removed. No further checks will be performed.

2. The `rules.conf` file will be parsed:

   * If there is a rule that matches the inserted USB device it will be applied. No further checks will be performed.

   * Otherwise it will go to the step 3.

3. From the `usbguard-daemon.conf` the value of `ImplicitPolicyTarget` will be checked:

   * If it is `allow` the device will be authorized

   * If it is `block` or `reject` the device will be, respectively, deauthorized or removed.


On top of that, from the CLI/DBus there is a function called `applyDevicePolicy` that can be used to override the decision that USBGuard took after the three steps listed above.
In `applyDevicePolicy` there is a parameter called `permanent`.
If it is set to `False` the policy override will just work as long as the device stays plugged in.
Otherwise if set to `True`, the policy will be stored in the `rules.conf` file.


### USBGuard limitations
USBGuard configuration can be accessed and manipulated with DBus.
Unfortunately there were two major deal breaker for us:

* There were no signals when a parameter were changed. In this way requiring a polling for getting the current updated configuration values.

* From DBus it was possible to get and set only the `InsertedDevicePolicy` parameter.

The first issue was solved with [this PR](https://github.com/USBGuard/usbguard/pull/259), now already merged in master.

The second is an issue for us because without being able to manipulate `ImplicitPolicyTarget` we can't, for example, easily reach a state where we allow every new USB devices.
Regarding this limitation I proposed a fix with [this PR](https://github.com/USBGuard/usbguard/pull/265), and it is currently waiting for approval.


## Work done so far
My starting point was regarding [the first step](https://wiki.gnome.org/Internships/2018/Projects/USB-Protection#Plan) where we display an entry in GNOME Control Center under the Privacy tab that let the user permanently allow or disallow new USB devices.
So far I created two proof of concepts of this functionality:

1. [The first](https://gitlab.gnome.org/denittis/gnome-control-center/commits/PoC_USB_menu) uses a GSettings entry to store the chosen value.

2. [The second](https://gitlab.gnome.org/denittis/gnome-control-center/commits/USB_menu_1) where the displayed value always reflects the `InsertedDevicePolicy` from the USBGuard configuration.

![first step](/assets/images/usb-first-step-1.gif)


For [the second step](https://wiki.gnome.org/Internships/2018/Projects/USB-Protection#Plan) the plan was to give an option to forbid new USB devices only when the lock screen was active.
This can be considered a sort of a middle ground between the permanently allow and disallow.
So far I did a [first proof of concept](https://gitlab.gnome.org/denittis/gnome-control-center/commits/usb_drop_menu) where I displayed a drop down menu letting the users choose between "always", "never" and "when lock screen is active".

The chosen property is then propagated to the USBGuard configuration and also stored with GSettings.


## What to expect in the following 1-2 weeks

Currently in the PoC that I realized I'm working exclusively with the `InsertedDevicePolicy` property.
While this is good enough when we want to block USB devices, it is subpar when we want to deactivate the protection.
The only way to be 100% sure that every new devices will be allowed is with `ImplicitPolicyTarget`.
As soon as we will be able to control it too, I'll adjust my implementation accordingly.
In the meantime I can prepend a wildcard "allow" to the rule file to achieve the same result.

I also need to improve the USBGuard check, in order to better inform the users if something is not working as expected (e.g. too old version of USBGuard or wrong initial USBGuard configuration).

Regarding the second step I'll create a patch for GNOME Shell that binds the USB locking logic to the lock screen process and, before acting, it checks the stored property from GNOME Control Center to decide how to behave based on the users choice.
In this step also an interesting question raises: What to do with devices which have been inserted while the lock screen was on?
Initially we probably keep them blocked and maybe we should show a notification about them to inform the users that they need to be re-plugged to make them work.


## Thanks

I'd like to thanks Tobias Muller for guiding me in this project and the whole GNOME foundation for this awesome opportunity.