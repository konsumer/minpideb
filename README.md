# minpideb

A truly minimal starting-point for pi embedded OS that you can still use apt.

# WIP

I am still working out ideas with this.

## Why not use X?

- [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/) - not nearly minimal enough. It contains a lot of comfortable tools & services, but is not optimized for fast boot
- [pi-gen](https://github.com/RPi-Distro/pi-gen) this is the tool used to make their Raspberry Pi OS disks. You can tune the stages down to much less, but the minimum is basically debootstrap, and I want even less. I took osme of their scripts, though, to use here. Great option if you want debootstrap + a few things.
- `debootstrap` - not minimal enough for me. It uses proper versions of things instead of busybox. It installs quite a few sensible, but maybe not needed things, and stiull requires more things to make it work well on pi.
- [buildroot](https://buildroot.org/) - it's possible to get things very minimal with this, and booting incredibly fast, but you ahve to either use their packages, or figure something else out. It seems much more suited to a truly embedded system, where you install everything on the image, then don't really mess with what is installed. This is a great option, if it fits your use-case, but I want to be able to install more stuff with apt, after the initial setup (for easier dev,a nd also for my users.)

## get started

There are a few things you need to make decisions about to get it working:

- a base qcow image - build with `./scripts/build`
- init - there is no proper init. You can use [the one in busybox](https://www.halolinux.us/embedded-systems/busybox-init.html) for a minimal, but still functional method, or you can even just add `init=` to your kernel params, or in the other direction, install systemd.
- networking - add packages for wifi/networking/etc, set hostname, edit resolv.conf
- install any other packages you need to get it booting how you want I will try to make recommmendations below.
- convert qcow to image for pi