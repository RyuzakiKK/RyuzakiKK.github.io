---
title:  "Initial Magic Wormhole implementation in Keysign"
date:   2017-06-17 15:00:00 +0100
category: GNOME
tags:
  - GNOME
---

I'm nearly at the end of the third week for this GSoC project and I'm happy to say that the planned Magic Wormhole implementation is now working.

### What is Magic Wormhole?
---
*Get things from one computer to another, safely.*

This package provides a library and a command-line tool named wormhole, which makes it possible to get arbitrary-sized files and directories (or short pieces of text) from one computer to another. The two endpoints are identified by using identical "wormhole codes": in general, the sending machine generates and displays the code, which must then be typed into the receiving machine.

### What is the workflow in Keysign with Wormhole?
---

Let's assume that Bob wants to sign the Alice's public key:
* Alice selects her key to send and a wormhole code will be generated.
* Alice communicates in a safe way the wormhole code to Bob.
* Bob enters the code in the receive tab and he automatically downloads the Alice's public key

If you are not familiar with GNOME-Keysign, this is exactly the same workflow that was used (and still is) when we want to transfer a key with a local Avahi server.
The only difference is that now are presented to the user both the avahi and wormhole codes.

![wormhole transfer](/assets/images/wormhole-transfer-1.gif)


### The security field in the receive tab accept both Avahi and Wormhole codes?
---

Yes.
Underneath we automatically check the type of the entered code and we use the correct service accordingly.
The wormhole code is always composed of a number, that identifies the channel used, and some english phonetically-distinct words.

### When should I use Avahi or Magic Wormhole?
---

| | LAN Only | Isolated LAN + Internet access | LAN + Internet access | Remote|
|-|:--------:|:------------------------------:|:---------------------:|:-----:|
|Avahi | ✔   |                                |           ✔           |       |
|Wormhole|   |                ✔               |           ✔           |   ✔   |

### Can I receive the public key with Magic Wormhole CLI version?
---
Yes.
The Keysign Magic Wormhole implementation is interoperable with the official command-line tool.
This means that Alice can send her key with Keysign and Bob can receive it using wormhole from command line.

![wormhole interoperability](/assets/images/wormhole-transfer-2.gif)

### What's next
---
In the following days I'll try to improve the error handling when for example a user receives a wrong/corrupted public key, or even inform the user if someone is trying to download the public key with the wrong Magic Wormhole code.

Also the wormhole unit tests need to be expanded and improved.
