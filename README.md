# overlayroot
mounts an overlay filesystem over the root filesystem

I use this for my Raspberry Pi, but it should work on any Debian or derivative.

The root file system on the sd-card is mounted read-only on /overlay/lower, and / is a
read-write copy on write overlay.

There are two sets of instructions below: [Raspbian](#raspbian) and [Ubuntu for ARM](#ubuntu-for-arm).

## Convinience Script for Armbian

Use enableoverlayroot script to automatically setup they same way as under Ubuntu for ARM.
This script has been tested on armbian.
```bash
./enableoverlayroot
```

Consider also adding `40-ram-filesystem` to `/etc/default/armbian-motd` so that the user is prompted that overlayroot is used when they login
```bash
cp 40-ram-filesystem /etc/update-motd.d/
```

## Raspbian

It uses initramfs. Stock Raspbian doesn't use one so step one would be to get initramfs working. 
Something like:

```bash
sudo mkinitramfs -o /boot/init.gz
```

Add to /boot/config.txt
```bash
initramfs init.gz
```

Test the initramfs works by rebooting. It should boot as normal.

Add the following line to /etc/initramfs-tools/modules
```
overlay
```

Copy the following files
- hooks-overlay to /etc/initramfs-tools/hooks/
- init-bottom-overlay to /etc/initramfs-tools/scripts/init-bottom/

install busybox
```bash
sudo apt-get install busybox
```

then rerun

```bash
sudo mkinitramfs -o /boot/init.gz
```
Now skip down to [all distributions](#all-distributions) to finish the installation.

## Ubuntu for ARM

Add the following line to /etc/initramfs-tools/modules
```
overlay
```

Copy the following files
- hooks-overlay to /etc/initramfs-tools/hooks/
- init-bottom-overlay to /etc/initramfs-tools/scripts/init-bottom/

install busybox-static
```bash
sudo apt-get install busybox-static
```

then run

```bash
sudo update-initramfs -k $(uname -r) -u
```

Now continue to [all distributions](#all-distributions) to finish the installation.

## all distributions

add contents of ./.bashrc to ~/.bashrc

```bash
cat .bashrc >> ~/.bashrc
```

After rebooting, the root filesystem should be an overlay. If it's on tmpfs any changes 
made will be lost after a reboot. If you want to upgrade packages, for example,
run `rootwork`, the prompt should prepend a warning sign indicating / is unprotected.
```bash
[⚠️ ] :#
```

You're now making changes to the sdcard, and changes will be permanent.

I use `rootwork` to work on the real root filesystem.
I put it in ~/bin and add ~/bin to my path.

The /run directory is problematic to umount, so atm `rootwork` --rbind mounts it
on the sd-card root file system, /overlay/lower, and it isn't umounted like /boot 
/proc /sys and /dev are.

After you've finished working on the sd-card run `exit`. `rootwork` tries to clean up 
by umounting all the mounts it mounted and remount /overlay/lower read-only, but 
often it can't due to an open file or something else causing the filesystem to be busy.
It's probably a good idea to reboot now for 2 reasons:
 
- leaving /overlay/lower read-write could cause file corruption on power loss. 
- to test it still boots ok after the changes you've just made.

Whenever the kernel is updated, for Raspbian you need to rerun 

```bash
sudo mkinitramfs -o /boot/init.gz
```

and for Ubuntu for ARM

```bash
sudo update-initramfs -k $(uname -r) -u
```

TODO: see if there's a hook to automatically run `sudo mkinitramfs -o /boot/init.gz` 
on kernel install

There are comments in some of the files you might want to read
and that's about it.
