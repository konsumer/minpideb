# minpideb

A truly minimal starting-point for pi embedded OS that you can still use apt & regular raspbian stuff.

I wanted a completely minimal pi OS, that I can still install stuff with `apt`, but not much else. It's more manual, and you will have to do stuff to get it how you want, but it's a good starting-point for me.

# WARNING

I actually moved on to [alpibase](https://github.com/konsumer/alpibase), which is a similar concept, but using alpine as the base, so busybox as a base works a bit better.


# WIP

I am still working out ideas with this.

## Why not use X, instead?

I like all of these, and have used them at various times, but chose to make my own. Here's why.

- [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/) - not nearly minimal enough for some thigns I am working on. It contains a lot of comfortable tools & services, but is not optimized for fast boot. It's a great choice, though, for like a proper CLI linux system.
- [pi-gen](https://github.com/RPi-Distro/pi-gen) this is the tool used to make their Raspberry Pi OS disks. You can tune the stages down to much less, but the minimum is basically debootstrap, and I want even less. I took some of their scripts, though, to use here. Great option if you want to make something that is basically one of their distros (lite, medium, or full) with a little modification.
- `debootstrap` - still not minimal enough for me. It uses proper versions of things instead of busybox. It installs quite a few sensible, but often not needed things, and still requires more things to make it work well on pi.
- [buildroot](https://buildroot.org/) - it's possible to get things very minimal with this, and booting incredibly fast, with a very small disk image, but you have to either use their packages, or figure something else out. It seems much more suited to a truly embedded system, where you install everything on the image, then don't really mess with what is installed. This is a great option, if it fits your use-case, but I want to be able to install stuff more easily with apt, after the initial setup (for easier dev, and also for my users.)


## get started

There are a few things you need to make decisions about to get it working. Here are the basic steps:

- generate a base qcow image - `./scripts/build`
- init - there is no proper init. You can use [the one in busybox](https://www.halolinux.us/embedded-systems/busybox-init.html) for a minimal, but still pretty functional method, or you can even just add `init=` to your kernel params, or in the other direction, install full-on systemd, in order to do regular startup stuff, more regular.
- networking - add packages for wifi/networking/etc, set hostname, edit resolv.conf
- install any other packages you need to get it booting how you want.
- convert qcow to regular SD image for pi


### example

Here i will make an example system, all ready to go. We'll just make something that is minimal, but can run inside qemu, and is functional enough to install things.

First, I will make my own project:

```sh
mkdir myos
cd myos
git init
echo "./out/" > .gitignore
git submodule add https://github.com/konsumer/minpideb.git
```

I like to record what I do in a script, if possible, so I can easily replay it, or just have some notes:

**build-myos.sh**

```bash
#!/bin/bash

if [ "$(id -u)" != "0" ]; then
    echo "Please run as root" 1>&2
    exit 1
fi

export MINIDEB_NAME="myos"

export WORK_DIR="${WORK_DIR:-$(realpath out)}"

source ./minpideb/scripts/qcow2_handling

./minpideb/scripts/build "${WORK_DIR}"
mount_qimage "${WORK_DIR}/image-${MINIDEB_NAME}.qcow2" "${WORK_DIR}/root"

cat << CHROOT | chroot "${WORK_DIR}/root" sh
apt-get update
apt-get upgrade -y
apt-get install -y netbase
CHROOT

sed "s/minipideb/${MINIDEB_NAME}/g" -i "${WORK_DIR}/root/etc/hostname"
sed "s/minipideb/${MINIDEB_NAME}/g" -i "${WORK_DIR}/root/etc/hosts"
umount_qimage "${WORK_DIR}/root"
```

Then run it:

```
chmod 755 build-myos.sh
sudo ./build-myos.sh
```
