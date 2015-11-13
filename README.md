# magic-trackpad-2-linux
Instructions for installing the Apple Magic Trackpad 2 on Linux

# Intro

I should start this by pointing out that I currently **do not** have a working configuration for the Magic Trackpad 2 on Linux. This is meant to document my experiments in hope that someone more experienced than me can jump in at some point and magically provide with a solution.

# My setup

I am running Ubuntu 14.04 LTS (trusty) with kernel `3.13.0-66-generic`.

# Out-of-the-box setup

Plugging the trackpad via USB does work to some extent, but only normal movements and left clicks are working. No multitouch, no right click, no scrolling, no option to change sensitivity exist.

# Synaptics drivers

The first thing I tried was to load the Synaptics drivers for the device.

With `lsusb`, I found out the USB ID of the device:

```bash
% lsusb
...
Bus 003 Device 018: ID 05ac:0265 Apple, Inc. 
...
```

I then used this to select the synaptic drivers for the input class, by adding the following lines to my `/etc/X11/xorg.conf`:

```
Section "InputClass"
  Identifier "Magic Trackpad"
  MatchUSBID "05ac:0265"
  Driver "synaptics"
EndSection
```

I then restarted X with `sudo restart lightdm` (note: it closes all your running programs, so make sure you know what your are doing).

After this, the trackpad stopped working entirely (not even basic movements and click), and I then looked into the X logs (`/var/log/Xorg.0.log`), and found the following:

```
TODO
```

Since the error message mentioned not being able to identify the protocol, I looked up the possible alternatives in `man synaptics`, which lists `auto-dev`, `event`, `psaux`, `psm`.

The following configuration gave me a different error:

```
Section "InputClass"
  Identifier "Magic Trackpad"
  MatchUSBID "05ac:0265"
  Driver "synaptics"
  Option "Protocol" "psaux"
EndSection
```

```
[389749.711] (II) config/udev: Adding input device Apple Inc. Magic Trackpad 2 (/dev/input/mouse3)
[389749.711] (**) Apple Inc. Magic Trackpad 2: Applying InputClass "Magic Trackpad"
[389749.711] (II) Using input driver 'synaptics' for 'Apple Inc. Magic Trackpad 2'
[389749.711] (**) Apple Inc. Magic Trackpad 2: always reports core events
[389749.711] (**) Option "Protocol" "psaux"
[389749.711] (**) Option "Device" "/dev/input/mouse3"
[389749.732] (--) synaptics: Apple Inc. Magic Trackpad 2: invalid x-axis range.  defaulting to 1615 - 5685
[389749.732] (--) synaptics: Apple Inc. Magic Trackpad 2: invalid y-axis range.  defaulting to 1729 - 4171
[389749.732] (--) synaptics: Apple Inc. Magic Trackpad 2: invalid pressure range.  defaulting to 0 - 255
[389749.732] (--) synaptics: Apple Inc. Magic Trackpad 2: invalid finger width range.  defaulting to 0 - 15
[389749.752] (EE) synaptics: Apple Inc. Magic Trackpad 2: Query no Synaptics: 6003C8
[389749.752] (--) synaptics: Apple Inc. Magic Trackpad 2: no supported touchpad found
[389749.752] (EE) synaptics: Apple Inc. Magic Trackpad 2: Unable to query/initialize Synaptics hardware.
[389749.788] (EE) PreInit returned 11 for "Apple Inc. Magic Trackpad 2"
[389749.788] (II) UnloadModule: "synaptics"
[389749.788] (II) config/udev: Adding input device Apple Inc. Magic Trackpad 2 (/dev/input/event17)
[389749.788] (**) Apple Inc. Magic Trackpad 2: Applying InputClass "evdev pointer catchall"
[389749.788] (**) Apple Inc. Magic Trackpad 2: Applying InputClass "Magic Trackpad"
[389749.788] (II) Using input driver 'synaptics' for 'Apple Inc. Magic Trackpad 2'
[389749.788] (**) Apple Inc. Magic Trackpad 2: always reports core events
[389749.788] (**) Option "Protocol" "psaux"
[389749.789] (**) Option "Device" "/dev/input/event17"
[389749.789] (--) synaptics: Apple Inc. Magic Trackpad 2: invalid x-axis range.  defaulting to 1615 - 5685
[389749.789] (--) synaptics: Apple Inc. Magic Trackpad 2: invalid y-axis range.  defaulting to 1729 - 4171
[389749.789] (--) synaptics: Apple Inc. Magic Trackpad 2: invalid pressure range.  defaulting to 0 - 255
[389749.789] (--) synaptics: Apple Inc. Magic Trackpad 2: invalid finger width range.  defaulting to 0 - 15
[389749.809] (EE) synaptics: Apple Inc. Magic Trackpad 2: Query no Synaptics: 000000
[389749.809] (--) synaptics: Apple Inc. Magic Trackpad 2: no supported touchpad found
[389749.809] (EE) synaptics: Apple Inc. Magic Trackpad 2: Unable to query/initialize Synaptics hardware.
[389749.840] (EE) PreInit returned 11 for "Apple Inc. Magic Trackpad 2"
[389749.840] (II) UnloadModule: "synaptics"
```

I also tried tapping into the raw event stream, and after playing around with byte sizes, I concluded that the events are 24 bytes each, and can be print like:

```bash
% sudo od -x -w24 /dev/input/event17
0000000 64d8 5646 0000 0000 d4e1 0007 0000 0000 0002 0000 ffff ffff
0000030 64d8 5646 0000 0000 d4e1 0007 0000 0000 0000 0000 0000 0000
0000060 64d8 5646 0000 0000 ffde 0007 0000 0000 0002 0000 fffe ffff
0000110 64d8 5646 0000 0000 ffde 0007 0000 0000 0000 0000 0000 0000
0000140 64d8 5646 0000 0000 2ad3 0008 0000 0000 0002 0000 ffff ffff
0000170 64d8 5646 0000 0000 2ad3 0008 0000 0000 0000 0000 0000 0000
0000220 64d8 5646 0000 0000 55cb 0008 0000 0000 0002 0000 ffff ffff
0000250 64d8 5646 0000 0000 55cb 0008 0000 0000 0000 0000 0000 0000
0000300 64d8 5646 0000 0000 da9a 0008 0000 0000 0002 0000 fffc ffff
0000330 64d8 5646 0000 0000 da9a 0008 0000 0000 0000 0000 0000 0000
0000360 64d8 5646 0000 0000 308a 0009 0000 0000 0002 0001 0002 0000
0000410 64d8 5646 0000 0000 308a 0009 0000 0000 0000 0000 0000 0000
0000440 64d8 5646 0000 0000 5b82 0009 0000 0000 0002 0001 0003 0000
0000470 64d8 5646 0000 0000 5b82 0009 0000 0000 0000 0000 0000 0000
0000520 64d8 5646 0000 0000 8a62 0009 0000 0000 0002 0001 0004 0000
0000550 64d8 5646 0000 0000 8a62 0009 0000 0000 0000 0000 0000 0000
0000600 64d8 5646 0000 0000 b559 0009 0000 0000 0002 0001 0005 0000
0000630 64d8 5646 0000 0000 b559 0009 0000 0000 0000 0000 0000 0000
0000660 64d8 5646 0000 0000 e051 0009 0000 0000 0002 0001 0004 0000
0000710 64d8 5646 0000 0000 e051 0009 0000 0000 0000 0000 0000 0000
0000740 64d8 5646 0000 0000 0b49 000a 0000 0000 0002 0001 0001 0000
0000770 64d8 5646 0000 0000 0b49 000a 0000 0000 0000 0000 0000 0000
0001020 64d8 5646 0000 0000 3642 000a 0000 0000 0002 0000 0001 0000
0001050 64d8 5646 0000 0000 3642 000a 0000 0000 0002 0001 0002 0000
0001100 64d8 5646 0000 0000 3642 000a 0000 0000 0000 0000 0000 0000
0001130 64d8 5646 0000 0000 6521 000a 0000 0000 0002 0000 0001 0000
0001160 64d8 5646 0000 0000 6521 000a 0000 0000 0002 0001 0002 0000
0001210 64d8 5646 0000 0000 6521 000a 0000 0000 0000 0000 0000 0000
0001240 64d8 5646 0000 0000 9019 000a 0000 0000 0002 0000 0001 0000
0001270 64d8 5646 0000 0000 9019 000a 0000 0000 0002 0001 0001 0000
0001320 64d8 5646 0000 0000 9019 000a 0000 0000 0000 0000 0000 0000
```

Let's try to break a single row down:
 - The first 16 bytes are probably just a timestamp (the second byte increments by one every second)
 - The next 8 bytes seem to represent the relative position of the cursor, basically the direction in which the finger is moving on the trackpad (positive towards the bottom and to the left of the surface of the trackpad).

Notably, events seem to only be produced when the finger is actually moving (not tapping or pressing), which suggests that the device is not even emitting events related to multitouch or pressure sensitivity at all.

At this point, my conclusion is that the more advanced functionalities are disabled in hardware, and only OSX knows the magic combination to unlock them. When plugged to any other computer, the trackpad will only send basic events.
