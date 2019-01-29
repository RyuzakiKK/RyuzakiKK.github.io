---
title:  "GNOME Security Internship - Update 4"
date:   2019-01-29 18:00:00 +0100
category: GNOME
tags:
  - GNOME
---

Here you can find the [introduction](/gnome/internship-preparation/), [the update 1](/gnome/internship-update-1/), [the update 2](/gnome/internship-update-2/) and [the update 3](/gnome/internship-update-3/).

## Graphical recap
After 4 long posts talking about USB devices, lock screen and keyboards are you a bit lost?
Are you trying to find an answer to the question: "What will happen when I plug a USB device?"

Don't worry, I'm here to help you with an easy to follow flowchart.

![USB flowchart](/assets/images/usb-flowchart.png)

## GNOME Control Center, USBGuard version check
In our implementation we use functionality that are available only from USBGuard 0.7.5.
In reality the last version of USBGuard is the 0.7.4, but the changes that we need are already in master.
So to be more precise, we require a version newer than 0.7.4.

Because USBGuard will most likely be an optional dependency, how do we check if the running version is new enough?
Given the fact that the new version of USBGuard has an additional get/set method for `ImplicitPolicyTarget`, I added a check to GNOME Control Center where I try to get this parameter.
If it fails it means that the running version of USBGuard is too old.

## The time has come. Merge Requests!
After a call with Georges Basile Stavracas Neto and Benjamin Berg we agreed that probably the best way forward was to create the required merge requests as early as possible.
Even if we will probably aim for an inclusion in the GNOME 3.34 cycle, doing it now will facilitate the gathering of feedback for the actual implementation and, maybe even more importantly, feedback for our design choices.

So this morning I prepared [the](https://gitlab.gnome.org/GNOME/gnome-control-center/merge_requests/366/) [required](https://gitlab.gnome.org/GNOME/gnome-shell/merge_requests/369) [merge](https://gitlab.gnome.org/GNOME/gsettings-desktop-schemas/merge_requests/15) [requests](https://gitlab.gnome.org/GNOME/gnome-settings-daemon/merge_requests/75).


## gnome-settings-daemon tests
I wrote a couple of tests for the USB daemon.
The first is about testing the configuration sync between gsettings and USBGuard.
The second checks if the "allow everything" rule gets added correctly to the USBGuard configuration.

They work but the code is still ugly.
So right now they are not included in the MR that I did.
After a cleaning and another correctness check I'll be able to do so.


## Progress on the second part, the keyboard's dangerous keys
Right now I added a keyboard protection entry in gsettings.
I'm not 100% sure if this will be the best place where to store it, but is good enough for starting.
And also, if necessary, it will be easy to change it later on.

I did a first implementation on Mutter and so far is it working as expected.
When we receive a key input event we check if the protection is active.
If it is we check if the pressed key is a dangerous one.
If that's also true we generate a sha256 hash starting from the device name, the vendor id, the product id and its serial number.

Then we will be able to match this hash against a whitelist/blacklist database.

I'm also working on presenting these information on control center.
The first step is a tab similar to the thunderbolt one, where we have a general toggle to enable/disable the USB keyboard protection and then a list of known devices with their current permission.

I'll try to not use too much time on this first implementation of control center because it will be more a proof of concept and a base to better understand how to store persistent devices information and how to pass them to mutter.

### And the performance?
While I wrote the key handling on mutter I asked myself: "and the performance?".
I was really curios about it, so I added a timer around the key handling method in mutter and I timed how much my additional check affected the performance.

* Mutter stock: 1-2 usec to compute the pressed key.
* With my modification with keyboard security off: 1-2 usec. It's just an extra if check.
* With keyboard security on: 1-2 usec. An extra if and a binary search against an int list of keys.
* Keyboard security on and you press a "dangerous" key: 7-12 usec. In this case we also need to compute the sha256 of the device.

For prospective, the best consumer monitors available right now have an input lag of 9-10ms, while an average monitor can take 30ms or more.
[This site](https://displaylag.com/display-database/) has been used as a source.

Keeping in mind that 1 millisecond (ms) is 1000 microseconds (us), I'm pleased to see that even when we need to generate a sha256 hash the performance hit is negligible.


## FOSDEM
This weekend I'll attend the FOSDEM event.
So if you want to talk a bit about this project I'll be there.

On Sunday Tobias Muller and me will [have a talk](https://fosdem.org/2019/schedule/event/usb_borne_attacks/) where we will present this project.


## What to expect next and comments
Hopefully by the next project update we will have a working first implementation of "limit keyboard capabilities" with all the required pieces together.

Feedback is welcome!