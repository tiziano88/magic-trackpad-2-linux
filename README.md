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

After this, the trackpad stopped working entirely (not even basic movements and click), and I then looked into the X logs, and found the following:

```
TODO
```

Since the error message mentioned not being able to identify the protocol, I then tried with various other alternatives for that option. Eventually, the following configuration gave me a different error:

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
