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
I use SDDM as a display manager.  In your ~/.local/src/dwl folder there should be a file called dwl.desktop.  During sudo make install clean this will be copied to /usr/local/share/wayland-sessions/dwl.desktop.  Make sure it looks like this to contain the dwl-startup.sh script.  The -s is the startup argument of dwl.
```
[Desktop Entry]
Name=dwl
Comment=dwm for Wayland
Exec=dwl -s ~/.local/dwl-startup.sh
Type=Application
```

## Terminal, application launcher and bar
As a terminal I use Kitty with background_opacity 0.7.  Zsh is my default shell and I'm a huge fan of the powerlevel10k zsh theme: https://aur.archlinux.org/packages/zsh-theme-powerlevel10k.
In i3 I've used rofi the longest time and I switched to wofi now.  I honestly don't even rememeber where I got the theme from but it's included in the repository as well.
The menubar is Waybar.  Because my DWL is IPC patched and Waybar's config has been modified for DWL I'm able to use actually use the tags.  It's a fairly default Waybar config except I changed the colors to blue.  This is less distracting in my opinion.

## Keyboard hacks for be-latin1 (Belgian AZERTY layout)
Being Belgian I'm using a be-latin1 AZERTY keyboard layout.  This is always a mess to change tags.  After some fiddling with 'wev' I was able to modify the necessary keys and use SHIFT+& for tag 1, SHIFT+é for tag 2, SHIFT+" for tag 3 etc.
In config.h:
```
/* keyboard */
static const struct xkb_rule_names xkb_rules = {
        /* can specify fields: rules, model, layout, variant, options */
        /* example:
        .options = "ctrl:nocaps",
        */
        .layout = "be",
};
```
and also
```
        TAGKEYS(          XKB_KEY_ampersand, XKB_KEY_1,                     0),
        TAGKEYS(          XKB_KEY_eacute, XKB_KEY_2,                         1),
        TAGKEYS(          XKB_KEY_quotedbl, XKB_KEY_3,                 2),
        TAGKEYS(          XKB_KEY_apostrophe, XKB_KEY_dollar,                     3),
        TAGKEYS(          XKB_KEY_parenleft, XKB_KEY_percent,                    4),
        TAGKEYS(          XKB_KEY_section, XKB_KEY_asciicircum,                5),
        TAGKEYS(          XKB_KEY_egrave, XKB_KEY_ampersand,                  6),
        TAGKEYS(          XKB_KEY_exclam, XKB_KEY_asterisk,                   7),
        TAGKEYS(          XKB_KEY_ccedilla, XKB_KEY_parenleft,                  8),
```
Enjoy fellow Belgians :)

## Screenshots
I know everyone scrolls immediately to see the screenshots.

