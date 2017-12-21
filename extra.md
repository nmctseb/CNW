# Linux tips & tricks
## Keyboard layout 
* Ofwel `dpkg-reconfigure keyboard-configuration`
* Ofwel rechtstreeks `/etc/defaults/keyboard` aanpassen:
```bash
XKBMODEL="pc105"
XKBLAYOUT="be"
XKBVARIANT=""
XKBOPTIONS=""

BACKSPACE="guess"
```
* Dan nieuwe config toepassen met `sudo setupcon`
* Voor de GUI:
  * `setxkbmap be`
  * `udevadm trigger –subsystem-match=input –action=change`

## Wachtwoord vergeten
* unplug power
* verwijder SD-kaart & steek in je PC
* voeg in `cmdline.txt` toe, **ALLES op dezelfde regel**: `rw init=/bin/bash`
* plaats SD, verbind HDMI & toetsenbord, start op

```bash
passwd <user>     # LET OP: querty/azerty!
sync
exec /sbin/init
```
* verwijder de toegevoegde items weer uit `/boot/cmdline.txt`
* reboot

## Kiosk mode
TODO