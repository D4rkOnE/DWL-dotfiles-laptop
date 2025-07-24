# Welcome to the .dotfiles of my Archlinux DWL setup

After 15 years on i3 it was time to migrate to a Wayland compositor.  I always loved the suckless  philosophy and that's why I chose DWL.

## What is DWL?
DWL is a compact, hackable compositor for Wayland based on wlroots. It is intended to fill the same space in the Wayland world that dwm does in X11, primarily in terms of functionality, and secondarily in terms of philosophy. Like DWM, DWL is:

* Easy to understand, hack on, and extend with patches
* One C source file (or a very small number) configurable via config.h
* Tied to as few external dependencies as possible

## How to install in Archlinux
Install a minimal version of Archlinux.  Git clone https://codeberg.org/dwl/dwl to your ~/.local/src folder.  Make && sudo make install clean.

## Patches
DWL, by default, is very spartan.  But its functionalities can be extended with the patches from https://codeberg.org/dwl/dwl-patches.  I only applied the following patches:
* gaps.patch from https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/gaps because I prefer small gaps between windows.
* focusdir.patch from https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/focusdir because I was used to that from i3.
* ipc.patch from https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/ipc because this provides an ipc for wayland clients to get and set DWL state. The ipc is intended for status bars like Waybar like I'm using.

## How to patch
From the ~/.local/src/dwl folder, enter the command: patch -p1 < /path/to/filename.patch.  Then make and finally sudo make install clean.  Login in DWL again to see the changes.

## DWL config modifications
DWL's configuration can be adapted by changing config.h and config.mk files in the ~/.local/src/dwl folder.  

## DWL's startup script
I'm well aware that an autostart patch exist for DWL but I prefer to use a startup script.  I put it it ~/.local/dwl-startup.sh and it it contains:
```
#!/bin/bash

# Kill already running dublicate process
_ps="waybar mako swaybg"
for _prs in $_ps; do
    if [ "$(pidof "${_prs}")" ]; then
         killall -9 "${_prs}"
    fi
 done

# Start our applications
swaybg --image ~/Pictures/flowers_night_blue_hd_fantasy_landscape.jpg &
mako &
waybar &
exec dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP=wlroots
```
