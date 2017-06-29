---
title:  "Magic Wormhole improvements in Keysign"
date:   2017-06-29 23:30:00 +0100
category: GNOME
tags:
  - GNOME
---

From the [last blog post](/_posts/2017-06-17-initial-wormhole-implementation.markdown) the following are the improvements that I have done so far.

## User experience improvements

#### Let the user choose between Avahi and Wormhole
----
Initially I displayed the Wormhole code alongside the security code for Avahi. Users were shown two codes but no clue as to which one was right to use. 

Because of this now only one code at a time is displayed.
Adding a switch button the users can declare if they want to transfer the key in a local only mode (Avahi) or with Internet connection (Magic Wormhole).

Before
![wormhole transfer](/assets/images/avahi-worm-before.png)

After
![wormhole transfer](/assets/images/avahi-worm-after.gif)

----
#### Inform users about slow or no connection 
----
When a user chooses to use Magic Wormhole the program tries to reserve a channel and get a code contacting the Wormhole server.
Because the users may not have a working Internet connection, or a very slow one, I added an infobar after a timeout of 10 seconds that inform the users about a possible problem with the connection.

![wormhole transfer](/assets/images/infobar.png)

----
#### Every unnecessary UI element adds complexity
----
Trying to achieve an user-friendly UI we decided to remove the buttons "redo", "ok" and "cancel" from the result page.
The users can still accomplish the same actions using only the "back" button in the top bar and we avoid to overwhelm the users with a lot of extra elements.

Before
![wormhole transfer](/assets/images/buttons-before.png)

After
![wormhole transfer](/assets/images/buttons-after.png)


## Magic Wormhole error handling
---
Now if an user tries to download a gpg key with a wrong wormhole code the program will automatically stop offering the key and the user will be informed of this failed attempt.
There are also other errors displayed, for example if the connection attempt fails.

![wormhole transfer](/assets/images/error-handling.png)

## Automated unit tests
---
I started to write some automated unit tests that utilize the python module "nose".
Right now they tests a wormhole transfer checking the key integrity after the download.

## Bug fixing
---
And in the end a lot of bug fixing.
