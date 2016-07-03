---
title: Screensaver
date: 2016-07-03 14:39:19
categories:
  - linux-dev
  - Graphics
tags:
  - Xorg
---

This document describes the screensaver work flow.

<!--more-->

# Summary #
Usually there are 2 stages for the process of screen saver.
* stage 1: the application to call the screensaver daemon to show the screensaver, such as a picture.
* stege 2: to shutdown the monitor, which will call the DDX from X server.

Below are screensaver applications:
* gnome-screensaver
* xscreensaver
Both of them will NOT call DDX driver xf86IsUnblank() as [DDX Design](http://www.x.org/releases/X11R7.7/doc/xorg-server/ddxDesign.html).
```
Returns TRUE when the screen saver mode specified by mode requires the screen be unblanked, and FALSE otherwise. The screen saver modes that require blanking are SCREEN_SAVER_ON and SCREEN_SAVER_CYCLE, and the screen saver modes that require unblanking are SCREEN_SAVER_OFF and SCREEN_SAVER_FORCER. Drivers may call this helper from their SaveScreen() function to interpret the screen saver modes.
```

Helper command:
* xset
DDX driver will be called.

There is a screen saver extension in X server.
* Currently I didn't find it is enabled during the work.
* If it works, the *(pScreen->screensaver.ExternalScreenSaver) will be called during dixSaveScreens().

Anyway, whether screen saver extension enabled or not, the related setting will be done in dixSaveScreens()
to check the type is SCREEN_SAVER_ON, SCREEN_SAVER_OFF or others.
* xset will go through this path.
* xscreensaver will only show the screensaver as application.

# gnome-screensaver #
It's default screensaver in Ubuntu system.

* The daemon will start as `root` when system boot
```
root      2095  0.0  1.2 562416 42520 ?        Sl   04:20   0:01 /usr/bin/gnome-screensaver --no-daemon
```
* gnome-screensaver command to show blank screen by default
```
Usage:
gnome-screensaver-command [OPTION...]

Help Options:
-h, --help           Show help options

Application Options:
--exit               Causes the screensaver to exit gracefully
-q, --query          Query the state of the screensaver
-t, --time           Query the length of time the screensaver has been active
-l, --lock           Tells the running screensaver process to lock the screen immediately
-a, --activate       Turn the screensaver on (blank the screen)
-d, --deactivate     If the screensaver is active then deactivate it (un-blank the screen)
-V, --version        Version of this application
```

# xscreensaver #
If you like, you can install xscreensaver, co-existing with the gnome-screensaver, or remove it.
* xscreensaver will call `libGL.so` to show the screensaver.
  * If loading libGL.so is failed, no screensaver will be shown.
* Install xscreensaver
```
$ sudo apt-get install xscreensaver

Optional:
$ sudo apt-get install xscreensaver-data-extra xscreensaver-gl-extra
```
* Start xscreensaver daemon as `non-root`
```
$ xscreensaver -nosplash &
```
* start configuration GUI and start daemon automatically.
```
$ xscreensaver-demo
```
* xscreensaver command to show the screensaver with random picture by default
```
usage: xscreensaver-command -<option>

This program provides external control of a running xscreensaver process.
Version 5.15, copyright (c) 1991-2008 Jamie Zawinski <jwz@jwz.org>.

The xscreensaver program is a daemon that runs in the background.
You control a running xscreensaver process by sending it messages
with this program, xscreensaver-command.  See the man pages for
details.  These are the arguments understood by xscreensaver-command:

-activate     Turn on the screensaver (blank the screen), as if the user
              had been idle for long enough.

-deactivate   Turns off the screensaver (un-blank the screen), as if user
              activity had been detected.
```

# xset #
```
usage:  xset [-display host:dpy] option ...

To control Energy Star (DPMS) features:
  -dpms      Energy Star features off
  +dpms      Energy Star features on
   dpms [standby [suspend [off]]]
        force standby 
        force suspend 
        force off 
        force on 
        (also implicitly enables DPMS features) 
        a timeout value of zero disables the mode 
For screen-saver control:
  s [timeout [cycle]]  s default    s on
  s blank              s noblank    s off
  s expose             s noexpose
  s activate           s reset
For status information:  q
```
* xset s activate
  * Turn off the monitor
```
dix_main
  Dispatch
    dixSaveScreens
      AMDGPUSaveScreen_KMS
        xf86IsUnblank
          crtc->funcs->dpms
          output->funcs->dpms
```
* xset s reset
  * Recovery the monitor
* xset dpms force off
  * Turn off the monitor, wait for keyboard or mouse to activate
```
dix_main
  Dispatch
    ProcDPMSForceLevel
      DPMSSet
        dixSaveScreens
          AMDGPUSaveScreen_KMS
            xf86IsUnblank
              crtc->funcs->dpms
              output->funcs->dpms
```

# Screensaver extension #

X server has a extension for screensaver.

* The initial function ScreenSaverExtensionInit() in Xext/saver.c
* By default it looks no screensaver extension in X server.
```
CreateRootWindow(ScreenPtr pScreen)
{
...
pScreen->screensaver.ExternalScreenSaver = NULL;
...
}
```

# Reference #
* https://systembash.com/how-to-turn-off-your-monitor-via-command-line-in-ubuntu/
* http://unix.stackexchange.com/questions/79240/starting-screensaver-from-terminal
* https://wiki.edubuntu.org/DebuggingScreenLocking/HowScreenLockingWorks
* http://askubuntu.com/questions/64086/how-can-i-change-or-install-screensavers
