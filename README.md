# Welcome to the .dotfiles of my Archlinux DWL setup

After 15 years on i3 it was time to migrate to a Wayland compositor.  I always loved the suckless  philosophy and that's why I chose DWL.  I decided to rice DWL and go for an overall blue theme.  Hello to #dwl on irc.libera.chat :)

## What is DWL?
DWL is a compact, hackable compositor for Wayland based on wlroots. It is intended to fill the same space in the Wayland world that dwm does in X11, primarily in terms of functionality, and secondarily in terms of philosophy. Like DWM, DWL is:

* Easy to understand, hack on, and extend with patches
* One C source file (or a very small number) configurable via config.h
* Tied to as few external dependencies as possible

## How to install in Archlinux
Install a minimal version of Archlinux.  Git clone https://codeberg.org/dwl/dwl to your ~/.local/src folder.  Make && sudo make install clean.  All my installed packages can be found in pacman.txt.

## Patches
DWL, by default, is very spartan.  But its functionalities can be extended with the patches from https://codeberg.org/dwl/dwl-patches.  I only applied the following patches:
* gaps.patch from https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/gaps because I prefer small gaps between windows.
* focusdir.patch from https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/focusdir because I was used to that from i3.
* ipc.patch from https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/ipc because this provides an ipc for wayland clients to get and set DWL state. The ipc is intended for status bars like Waybar like I'm using.

## How to patch
From the ~/.local/src/dwl folder, enter the command: patch -p1 < /path/to/filename.patch.  Then make and finally sudo make install clean.  Login in DWL again to see the changes.
You can check beforehand with patch -p1 --dry-run < /path/to/filename.patch.

## DWL config modifications
DWL's configuration can be adapted by changing config.h and config.mk files in the ~/.local/src/dwl folder.  Modifying the C-code can be challenging to begin with. My changes and all keybinds are included in dotfiles.tar.

## DWL's startup script
I'm well aware that an [autostart](https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/autostart) patch exists for DWL but I prefer to use a startup script.  I put it in ~/.local/dwl-startup.sh and it contains:
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
mako -c ~/.config/mako/config &
waybar &
udiskie --tray &
nm-applet --indicator &
wl-clip-persist --clipboard regular &
wl-paste --type text --watch cliphist store &
wl-paste --type image --watch cliphist store &
swayidle -w timeout 300 'swaylock --screenshots --clock --effect-blur 7x5 -f' &
exec dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP=wlroots
```
As you can see swaybg is used to set the wallpaper.  I included a few wallpapers in this repository which I like a lot. 
I use SDDM as a display manager.  In your ~/.local/src/dwl folder there should be a file called dwl.desktop.  During sudo make install clean this will be copied to /usr/local/share/wayland-sessions/dwl.desktop.  Make sure it looks like this to contain the dwl-startup.sh script.  The -s is the startup argument of dwl.
```
[Desktop Entry]
Name=dwl
Comment=dwm for Wayland
Exec=dwl -s ~/.local/dwl-startup.sh
Type=Application
```

## Terminal, application launcher, file manager and bar
As a terminal I use Kitty with background_opacity 0.7.  

Zsh is my default shell and I'm a huge fan of the powerlevel10k zsh theme: https://aur.archlinux.org/packages/zsh-theme-powerlevel10k.

In i3 I've used rofi the longest time and I switched to wofi now.  The nord theme from https://github.com/joao-vitor-sr/wofi-themes-collection is used.

My favorite file manager is Thunar.  I'm a fan of aur/arc-gtk-theme which integrates well into this overall blue themed DWL.

The menubar is Waybar.  Because my DWL is IPC patched and Waybar's config has been modified for DWL I'm able to actually use the tags.  It's a fairly default Waybar config except I changed the colors to blue.  This is less distracting in my opinion.

## NetworkManager and nm-applet
Taking my laptop to work, home, hotels etc it's fun to easy switch between wifi networks.  I installed NetworkManager and enabled it with ```systemctl enable --now NetworkManager```.  Then I installed nm-applet and added ```nm-applet --indicator &``` to my dwl-startup.sh.  It adds a tiny networking applet in the tray section of Waybar and allows you to switch wifi networks easily.

## Wayland clipboard tool
I installed wl-clip-persist.  That installs 2 binaries: wl-paste and wl-clip-persist.  The following were added to my dwl-startup.sh file:
```
wl-clip-persist --clipboard regular &
wl-paste --type text --watch cliphist store &
wl-paste --type image --watch cliphist store &
``` 
## Screenshot tool
I installed sway-screenshots to be able to make screenshots.
```
static const char *ssoutputcmd[] = { "sway-screenshot", "-m", "output", "-o", "Pictures/screenshots", NULL };  /* screenshot of full screen */
static const char *ssselectioncmd[] = { "sway-screenshot", "-m", "region", "-o", "Pictures/screenshots", NULL }; /* screenshot of selection */
        { MODKEY,                    XKB_KEY_Print,      spawn,          {.v = ssoutputcmd} },
        { MODKEY|WLR_MODIFIER_SHIFT, XKB_KEY_Print,      spawn,          {.v = ssselectioncmd} },
```
Pressing MODKEY + PrtSc gives me a screenshot of my entire screen.  MODKEY + SHIFT + PrtSc allows me to select a region on my screen using the mouse.  Screenshots are saved to my ~/Pictures/screenshots folder.

## Auto-mount USB devices like external hard drives
I prefer my USB connected external hard drives to be mounted automatically. For this I installed udisks2 which is then enabled at boot via ```systemctl enable --now udisks2.service```. As for automount I installed udiskie which is autostarted with ```udiskie --tray &``` in my dwl-startup.sh script.  External USB devices will now be auto-mounted and accessible via Thunar.

## Mako notification daemon
Mako is also started via the dwl-startup.sh script with ```mako -c ~/.config/mako/config &```.  I added the following to ~/.config/mako/config:
```
default-timeout=10000

[urgency=high]
ignore-timeout=1
```
This means that notifications dissapear after 10 seconds, but urgent messages remain visible.  The default blue of mako matches well with the overall blue theme in DWL I'm going for anyways.

## Keyboard hacks for be-latin1 (Belgian AZERTY layout)
Being Belgian I'm using a be-latin1 AZERTY keyboard layout.  This is always a mess to change tags.  After some fiddling with 'wev' I was able to modify the necessary keys and use MODKEYT+& for tag 1, MODKEY+é for tag 2, MODKEY+" for tag 3 etc.
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

## Lockscreen configuration, shutdown monitor, hibernate and suspend.
### Lockscreen
I'm using a combination of swayidle and swaylock for this.  Install the following packages: swayidle, swaylock-effects-git (from AUR) and wlr-randr. My dwl-startup.sh script contains the following line: ```swayidle -w timeout 300 'swaylock --screenshots --clock --effect-blur 7x5 -f' &```.  This means that swayidle will launch swaylock after 5min (= 300 secs) by showing a blurred version of my then-current workspace.  The blurred screenshot version of swaylock is part of swaylock-effects-git.  Swaylock installed through pacman does not have the screenshots functionality.  
CTRL+ALT+L will immediately lock the screen.
```
static const char *lockcmd[] = { "swaylock", "--screenshots", "--clock", "--effect-blur", "7x5", "-f", NULL };
        { WLR_MODIFIER_CTRL|WLR_MODIFIER_ALT,            XKB_KEY_l,          spawn,          {.v = lockcmd} },
```
### Shutdown monitor
The wlr-randr command is used as a keyboard shortcut to shutdown the monitor.  The general syntax is: wlr-randr --output DISPLAY --off/--on.
```
static const char *screenoncmd[] = { "wlr-randr", "--output", "eDP-1", "--on", NULL};
static const char *screenoffcmd[] = { "wlr-randr", "--output", "eDP-1", "--off", NULL};
        { MODKEY|WLR_MODIFIER_CTRL,  XKB_KEY_o,          spawn,          {.v = screenoncmd} },
        { MODKEY|WLR_MODIFIER_CTRL,  XKB_KEY_a,          spawn,          {.v = screenoffcmd} },
```
### Hibernate
Hibernation requires some manuel work.  First of all, create a large swap file.  I'm using BTRFS so it's easy: ```btrfs filesystem mkswapfile --size 16g --uuid clear /swap/swapfile```.  Then ```swapon /swap/swapfile```.  
Add the following line to /etc/fstab: ```/swap/swapfile none swap defaults 0 0```.  Reboot now and see if your 16 GB swap is active after reboot.  
Now the initramfs needs modifications as well.  Edit file /etc/mkinitcpio.conf and look for the HOOKS line.  It should look like HOOKS=(base udev ...).  Add the word ```resume``` between brackets so it looks like this: HOOKS=(base udev ... resume).  
Now regenerate your initramfs by entering ```mkinitcpio -P```.  Reboot again and see if ```systemctl hibernate``` works.  
``` 
static const char *hibernatecmd[] = { "systemctl", "hibernate", NULL};
        { MODKEY|WLR_MODIFIER_CTRL,  XKB_KEY_h,          spawn,          {.v = hibernatecmd} },
```
I'm happy to report that all hardware, including wifi, is able to resume after hibernation.  This was a lot messier 10-15 years ago in Linux. 

### Suspend
Very simple, this can be achieved by using ```systemctl suspend``` and should work out of the box.
```
static const char *suspendcmd[] = { "systemctl", "suspend", NULL};
        { MODKEY|WLR_MODIFIER_CTRL,  XKB_KEY_s,          spawn,          {.v = suspendcmd} },
```
### SDDM theme
A goodlooking matching SDDM theme was found on https://github.com/sniper1720/elegant-sddm-archlinux-theme.  Git clone it, save it to /usr/share/sddm/themes/elegant-archlinux.  Copy /usr/lib/sddm/sddm.conf.d/default.conf to /etc/sddm.conf and change it to
```
[Theme]
Current=elegant-archlinux
```

### Neovim
There are many good neovim tutorials out there.  I'm using the following plugins: Lazyvim, nvim-telescope, nvim-treesitter.  https://dev.to/zt4ff_1/effective-neovim-setup-a-beginners-guide-1i81 is a wonderful guide to get you started.

## Screenshots
I know everyone scrolls immediately to see the screenshots.
Clean:
![Clean screenshot](https://github.com/himselfish/DWL-dotfiles-laptop/blob/main/clean.png)

Fake busy:
![Fake busy screenshot](https://github.com/himselfish/DWL-dotfiles-laptop/blob/main/busy.png)

Fake busy:
![Fake busy screenshot](https://github.com/himselfish/DWL-dotfiles-laptop/blob/main/busy2.png)



