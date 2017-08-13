---
title:  "GSOC Keysign Bluetooth update and GUADEC 2017"
date:   2017-08-13 16:00:00 +0100
category: GNOME
tags:
  - GNOME
---

## Bluetooth

With Bluetooth I ended up implementing two different ways to exchange keys because beforehand it was not clear which one of them was better.

#### Use BT discovery and change BT name
----

With this approach, after a key has been selected, the Bluetooh name will be changed to the key fingerprint.
This is like how Avahi works because we create a local server with the name of the fingerprint.

In the receiving side the user needs to enter the fingerprint, then a Bluetooth discovery will start searching for the right device.

We need to start a discovery because in order to establish a Bluetooth connection we need to know the MAC address of the device to connect to. 

These were in short the needed steps.

PROS:

  * No extra codes, we use the same key fingerprint already used with Avahi

CONS:

  * Need to change every time the BT name, a bit invasive approach
  * Relatively slow, the discovery to get the MAC address is not fast 

----
#### Use the BT MAC address directly
----

Instead of using the key fingerprint, another way is using the Bluetooth MAC address.
In this way the receiver will already knows who to connect to, and we can avoid to perform a discovery.

The problem is that now we have yet another code to display.

After some discussions I ended up with the choice of only embed the Bluetooth MAC address (the Bluetooth code) in the QR code. Doing so we can continue to display to the user always only one code.
This has the downside to limit the Bluetooth use exclusively to QR code, but the reason behind it is also that the QR code is safer than manually entering the code (I'll explain that later), so this is even an attempt to push the use of the QR.

PROS:

  * Faster, because we don't need to start a discovery
  * Don't need to change the BT name

CONS:

  * Need an extra code (BT MAC)

----
#### Considerations
----

Nothing is still definitive, but after some testing and discussions probably the second method is the preferred one.

----
#### Security
----

A transfer with Bluetooth could happen with or without pairing.

If two devices are paired the communication is encrypted, insted if they are not the communication is in plain text.
Even if the encryption was a good extra feature, I avoided to add the pairing to Keysign because it required additional user interaction and also pairing a new device only for a single connection was a bit overkill. Also if afterwards the user doesn't remember to delete the paired device it may even become a security problem.

So how can we be sure that the downloaded key has not been tampered?
With an hash-based message authentication code (HMAC)
In the QR code we embed a message authentication code that was also used for check the downloaded key with Avahi.
In this way we can reutilize this mechanism also for Bluetooth and be reasonably safe that the received key has not been altered.


----
#### Presenting the Bluetooth option
----

I used the least intrusive way for the user: Bluetooth, if available, is automatically added to the exsisting transfer methods.

The advantage is that this approach requires no extra steps for the user and no extra buttons in the GUI.


## Magic Wormhole

----
#### Refactoring to Inline Callbacks
----

Previously I implemented Magic Wormhole using the callback mechanism offered by Twisted.

After some discussions I decided to refactor the code to use the inline callbacks.
The advantages are that the code flow is now more linear and easier to follow and maintain.


## GUADEC 2017
--

This was my first GUADEC and I must say that it was amazing!
The conference gave me the opportunity to talk to very nice people and attend beautiful talks.
There were also wonderful social events! (like the 20th GNOME birthday and the walking tour for example)

I think that the talks that I liked most were:

  * _The GNOME Way (Allan Day) and The History of GNOME (Jonathan Blandford)_: As a newcomer to the GNOME development, these two talks helped me to **really** understand the principles of the GNOME community.
  * _Keynote: The Battle Over Our Technology (Karen Sandler)_: Very passionate and informative talk. Unfortunately explaining the importance of Free Software to other people is not easy.
  * _Resurrecting dinosaurs, what can possibly go wrong (Richard Brown)_: I didn't expect this kind of talk. He expressed some concerns and objections regarding Flatpak and the other application technologies. This talk made me think a lot about the current state, and what we should do to improve the situation.



I’d like to thank the GNOME Foundation for sponsoring me. This was a fantastic experience and I hope that we can met again next year in Almería.

![gnome badge](/assets/images/sponsored-badge-simple.png)
