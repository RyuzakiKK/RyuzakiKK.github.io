---
title:  "GNOME Security Internship - The end?"
date:   2019-02-28 14:15:00 +0100
category: GNOME
tags:
  - GNOME
---

Here you can find the [introduction](/gnome/internship-preparation/), [the update 1](/gnome/internship-update-1/), [the update 2](/gnome/internship-update-2/), [the update 3](/gnome/internship-update-3/), [the update 4](/gnome/internship-update-4/) and [the update 5](/gnome/internship-update-5/).


## The end? Is the internship already ended?

Yes, incredibly these three months went by so fast.
It was an awesome experience, so I'd like to thank the whole GNOME Foundation for giving me this opportunity.

## What's the project status?

The first part regarding protecting the system from potentially unwanted new USB devices can be considered completed.
Probably now it will requires just bug fixing and minor changes, if necessary.
[The](https://gitlab.gnome.org/GNOME/gnome-control-center/merge_requests/366/) [required](https://gitlab.gnome.org/GNOME/gnome-shell/merge_requests/369) [merge](https://gitlab.gnome.org/GNOME/gsettings-desktop-schemas/merge_requests/15) [requests](https://gitlab.gnome.org/GNOME/gnome-settings-daemon/merge_requests/75) are up.

The second part regarding limiting the number of usable keys for untrusted keyboards reached a working stage.
However it's still under evaluation which is the best way to achieve it, because even if with the current solution works it doesn't mean that this is the desirable way to do it.

## What will happen from tomorrow?

Even if the internship will be officially ended I'm still pledged to carry on these two features until they'll be merged in upstream.

Right now I'm actively searching for a job (preferably in the open source world), so in the next few days I'll still be able to spend a considerable amount of time in this project, at least until I find a full time job.

## GNOME Control Center USB protection switch

Old:
![G-c-c old USB popup](/assets/images/g-c-c-usb-switch.png)

New:
![G-c-c new USB popup](/assets/images/g-c-c-usb-new-switch.gif)

As I mentioned in the last blog post we realized that the always on USB protection was marginally better (or worse) than the protection with lock screen.
So we decided to leave in Control Center only an on/off switch that by default controls the USB lock screen protection.

If necessary it's still possible to enable the always on protection editing the `usb-protection-level` desktop schema.

This UI change allowed us to remove the ambiguous protection on switch with the drop down protection level "never block".

Now when you disable the protection from Control Center, in USBGuard we set InsertedDevicePolicy to apply-policy and we add an allow everything in the rules file.
In this way we try to leave USBGuard in a clean state.
From there USBGuard will never block USB devices anymore.
You are then free to leave it as is or, if you want a stringent or different protection behaviour, use a third parties GUI or even write your own script to handle USBGuard.

## Limit untrusted USB keyboards

<video controls> <source src="/assets/images/hwdb-g-c-c.webm" type="video/webm" /> </video>

Finally in this front we reached a first working implementation.

In the new Control Center tab we show an on/off switch and a list of currently plugged in keyboards.
This list is automatically updated when keyboards gets added or removed.

We store the authorization property with hwdb.
In this way we can bound it to a specific device product and retrieve this information directly from libinput.

When a new keyboard is added it is limited by default until you manually set it to be fully trusted.

While this implementation is working as expected, we are currently evaluating other alternatives.

For example in mutter every time we receive a new keystroke we check if it is a dangerous key.
As an alternative we could use the kernel `EVIOCSMASK` instead.

Another thing that we are evaluating is if we should replace the hwdb with another db created ad-hoc for this purpose.


## What to expect next and comments

In a couple of weeks I'll return hopefully with a few interesting updates regarding the untrusted USB keyboards.

Feedback is welcome!
