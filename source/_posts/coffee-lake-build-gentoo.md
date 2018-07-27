---
title: Minor details and fixes for a Coffee Lake build on Gentoo Linux
date: 2018-07-16 14:45:45
tags:
---

CPU: Intel i7-8700k

Cooler: Cryorig H5 Ultimate

Mobo: MSI z370 A-pro

RAM: GSkill RipJaws V 16gb @ 3000 MHz

Disk: Samsung EVO 970 M.2 NVMe 250

Case: NZXT H500

GPU: Nvidia 1080 GTX (GALAX, OC SNPR dual fans, white)

OS: Gentoo Linux

Cooler installation
-------------------

The H5 is tricky to install, preferably watch the youtube installation video
before attempting. Thermal compound is provided, don't get an extra tube unless
you need it.

Monitor temps after you install it - mine idles at 22-30C, the room ambient is
22-25C.

Motherboard specific stuff
--------------------------

Installation: the NZXT case does not provide all 8 (?) mounting screws for the
mobo, but that is OK. When I installed it I slightly bent a catch on the
backplate for the IO board, but no biggies.

Save settings and boot is F10.

RAM speed most likely will be detected wrong. It can be changed by going to BIOS
and to the OC section (toggle advanced with F7). 

Change boot mode to UEFI instead of Legacy + UEFI if you are planning on booting
from that.

IOMMU (VT-d feature) is disabled on this board by default. Turn it on in the OC
section -> CPU features. This processor supports it.

Caveat: This motherboard does *not* come with A LOT of USB ports, and I can only
find 1 host controller. This may be an annoyance if you want to pass an entire
host controller through to the guest VM in passthrough.

When booting in UEFI mode its supposed to look at `$ESP/EFI` but some
manufacturers get it wrong apparently. When doing grub install we need to add
`--removable`.

```
grub-install --target=x86_64-efi --efi-directory=/boot --removable
```

Motherboard display stuff

Advanced -> Integrated Graphics Configuration
Initiate Graphics Adapter -> Set to PEG (PCIe first)
IGD multi monitor support -> Enable (to enable the i915 chip)


Set gentoo-sources to use ~amd64 keyword then emerge to get (at the time of
writing) 4.17.6. This will load i915 automatically for the integrated GPU.

If you are on an older kernel version you need `i915.alpha_support=1` in your
kernel boot parameters (edit grub.cfg).

NVMe Disk
---------

Build in the NVMe driver to the kernel (*not* as a module).

```
CONFIG_NVME_CORE=y
CONFIG_BLK_DEV_NVME=y
```

More disk
---------

Enable ZFS. Note that the wiki is slightly out of date 

just install zfs-kmod and zfs.


```
echo "=sys-fs/zfs-kmod-9999 **" >> /etc/portage/package.accept_keywords/zfs-kmod
echo "=sys-fs/zfs-9999 **" >> /etc/portage/package.accept_keywords/zfs 
# when upgrading kernel
# after eselect kernel set n
# run make prepare in /usr/srx/linux
env EXTRA_ECONF='--enable-linux-builtin' \
    ebuild /usr/portage/sys-fs/zfs-kmod/zfs-kmod-9999.ebuild clean configure
(cd /var/tmp/portage/sys-fs/zfs-kmod-9999/work/zfs-kmod-9999/ && ./copy-builtin /usr/src/linux)
```

Then enable ZFS kernel module `CONFIG_ZFS` and build.

Then rebuild the nvidia driver just in case

`emerge -a @module-rebuild`

CPU Governor
------------

Set to `performance` and enable the intel pstate driver (default).

SSH
---

Edit the SSHD config and remove `AcceptEnv`. I always SSH from MacOS machines
and we do not want to be setting env like `LC_CTYPE=UTF-8` because glibc does not
support that by default.

Change authentication to publickey only, copy key over and restart sshd.

```sh
ssh alex@trixie '>> .ssh/authorized_keys' < id_rsa.pub
```

Current monitor setup
---------------------

```
                                +--------------+    +--------------+
                                |  2560x1440   |    | 2560x1440    |
                                |  MST on      |    | MST off      |
                                |  DP1.2       |    |              |
                                |              |    |              |
                                +-----+--------+    +----+---------+
                             mDP in|  |     |DP out      |  DP in
                                   |  |     |            |
                                   |  |     |            |
                                   |  |     |            |
+------------+DP out               |  |     +------------+
|  Intel     +---------------------+  |
|  Onboard   |                        |
|            |                        |
+------------+                        |
                                      |
+------------+        DP              |
|            +------------------------+
| GTX 1080   |
|            |
+------------+
```

In make.conf when setting `VIDEO_CARDS` set `nvidia intel i965`


Plan -

  * When system boots it boots off the 1080 as the boot GPU
  * Xorg starts on 1080 with i3 as the wm, puts workspace 2 on the second
  monitor
  * When we need to unbind, we kill X, start X on i915, workspace 2 moves over
  * PCIe unbind, rebind into VM


Create an intel xorg config

added this to startx

```
if [ -f /tmp/vm.lock ]; then
        serverargs="-config intel.xorg.conf"
fi
```

and intel.xorg.conf
```
Section "Device"
        Identifier  "intel"
        Driver      "modesetting"
        BusID       "PCI:0:2:0"
EndSection
```

# Controlling the displays

Install ddccontrol to use the DDC channels over DP

this DOES NOT WORK over nvidia GPUs (modern ones anyway) over DP (but it is
known to work over HDMI). 

Here's what you need to do: enable i2c support:

https://wiki.gentoo.org/wiki/I2C (enable everything in this wiki)

then run

```
ddccontrol -p -v
```

to probe

then for DP

```
ddccontrol -r 0x60 -w 15 dev:/dev/i2c-3 #DP
ddccontrol -r 0x60 -w 16 dev:/dev/i2c-3 #mDP
```

source: https://askubuntu.com/questions/860761/ubuntu-command-line-to-change-input-source-on-a-display-monitor
