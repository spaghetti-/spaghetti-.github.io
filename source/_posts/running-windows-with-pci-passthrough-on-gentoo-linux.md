---
title: Running Windows with PCI passthrough enabled on Gentoo Linux
date: 2018-07-27 12:23:18
tags:
 - linux
 - gentoo
 - nvidia
 - pci-passthrough
 - qemu
---

# Why?

* I have a 1080 GTX that I use to power my 2 1440p screens, render high
definition video, re-encoding and also to play Dota 2 on Linux.
* I also foresee the card running CUDA specific stuff just out of personal
interest.
* I _hate_ rebooting machines (the box is also a ZFS datastore that is accessible
over the network, music server, etc.).
* I want to play certain games on Windows (GTA V, Dying Light, etc.)

# Hardware

The requirements for pci passthrough can be found all over the internet,
particularly in the Arch wiki, but in short: If you've an Intel processor it
will need to support VT-d and IOMMU. Most modern processors support that. The
QEMU wiki states that most K processors don't - but the 8700k does support it.

The motherboard needs to support it as well (and again, most modern boards do)
  but some manufacturers are known to disable it to shave cost. Google around.

The MSI z370-A PRO board supports it. Couldn't find it in a database somewhere
but it does (tested).

I don't want to have two separate KB and mouse because that beats the point of
what I'm trying to achieve. I want a Linux host that can run games at times and
sorta 'alt-tab' into each other. However, if you care about achieving bare
metal latencies - you'd need to get a PCI USB hub that can be passed through.

The z370-A PRO motherboard comes with only 1 xHCI USB hub.

The rest of this post is tailored to hardware I own so

| Components                            |
| ------------------------------------- |
| Intel i7 8700k                        |
| MSI z370-A PRO                        |
| G.Skill Ripjaws V 16GB @ 3000MHz CL15 |
| Corsair 128GB SATA3 SSD               |
| Generic HID Keyboard                  |
| Generic HID Mouse                     |

The SSD is from another computer that no longer exists and was scavenged.

It is important that there's two GPUs available on the system. The i7 8700k
comes with its integrated graphics and that works fine.

# BIOS Setup

Enable VT-d in Overclocking (smh) -> CPU features. I have no idea why it's not
enabled by default on this motherboard.

Set the boot GPU to the Intel i915 by navigating to Advanced -> Integrated
Graphics Configuration -> Set to IGD.

When I boot from the 1080 GTX everything still works, but I can swap X between
nvidia and intel gpus exactly _two_ times. Any more and it'll stall the CPU and
I have to REISUB the machine. Details can be found in the email I sent to the
vfio-users mailing list [here](https://www.redhat.com/archives/vfio-users/2018-July/msg00005.html).

# Kernel setup

I'm on gentoo-sources 4.17.9 at the time of writing. You need to keyword it so
you'd need

```
sys-kernel/gentoo-sources ~amd64
```

Enable the following for IOMMU

```
CONFIG_IOMMU_SUPPORT
CONFIG_INTEL_IOMMU
CONFIG_INTEL_IOMMU_SVM
CONFIG_IRQ_REMAP
CONFIG_VFIO
CONFIG_VFIO_PCI
CONFIG_VFIO_PCI_VGA
```

I'll also upload my kernel configuration so that it's easier to reference.


Then add to `/etc/default/grub` and reboot

```
GRUB_CMDLINE_LINUX="iommu=on intel_iommu=on"
```

Check if the options work by checking the log

```
alex@trixie ~ $ dmesg | grep -e IOMMU -e DMAR
[    0.000000] ACPI: DMAR 0x000000007C5A8880 0000A8 (v01 INTEL  EDK2     00000001 INTL 00000001)
[    0.000000] DMAR: IOMMU enabled
```

Also enable the relevant Kernel flags for the Intel GPU and NVIDA GPU (see the
gentoo handbook for this).

The i965 (newer i915 name in mesa) seems to flicker on my screen and cause
artifacts at times. The Arch wiki recommends adding `i915.enable_psr=0` to get
around that. You can find more details about that here.

# Display setup

I changed my display setup slightly to remove DP chaining. The final setup is
like so

```

                                +--------------+    +--------------+
                                |  2560x1440   |    | 2560x1440    |
                                |  MST on      |    | MST off      |
                                |  DP1.2       |    |              |
                                |              |    |              |
                                +-----+--------+    +----+---------+
                             mDP in|  |      DP out      |  DP in
                                   |  |                  |
                                   |  |                  |
                                   |  |                  |
+------------+DP out               |  |                  |
|  Intel     +---------------------+  |                  |
|  Onboard   |                        |                  |
|            |                        |                  |
+------------+                        |                  |
                                      |                  |
+------------+        DP              |                  |
|            +------------------------+                  |
| GTX 1080   |                                           |
|            +-------------------------------------------+
+------------+
```

I removed the DP chaining because if you're (and you should be) using the nvidia
proprietary drivers on Linux, there's a bug preventing DDC control from working
over DP.

DDC control enables us to switch the inputs of the monitors from mDP <-> DP from
software (i2c write). It works fine with the intel GPU which works out well for
us since that's the display that'll be running X when we're in passthrough.

There's a nice little utility called `ddccontrol` that enables us to use this.

```
=app-misc/ddccontrol-db-20061014_p20121105 ~amd64
=app-misc/ddccontrol-0.4.2_p20140105-r2 ~amd64

emerge -a ddccontrol
```

# IOMMU Groups

And IOMMU group is the smallest set of devices the VM can grab at a time.

You can check your IOMMU groups by running this little command that I got from
many sources (Archwiki, /vg/s installgentoo page)

```sh
for iommu_group in \
      $(find /sys/kernel/iommu_groups/ -maxdepth 1 -mindepth 1 -type d); do \
  echo "IOMMU group $(basename "$iommu_group")"; \
  for device in $(ls -1 "$iommu_group"/devices/); do \
    echo -n $'\t'; lspci -nns "$device"; \
  done; \
done
```

Mine are

```
IOMMU group 7
        00:1c.0 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #1 [8086:a290] (rev f0)
IOMMU group 5
        00:16.0 Communication controller [0780]: Intel Corporation 200 Series PCH CSME HECI #1 [8086:a2ba]
IOMMU group 3
        00:08.0 System peripheral [0880]: Intel Corporation Xeon E3-1200 v5/v6 / E3-1500 v5 / 6th/7th Gen Core Processor Gaussian Mixture Model [8086:1911]
IOMMU group 11
        03:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 15)
IOMMU group 1
        00:01.0 PCI bridge [0604]: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor PCIe Controller (x16) [8086:1901] (rev 07)
        01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP104 [GeForce GTX 1080] [10de:1b80] (rev a1)
        01:00.1 Audio device [0403]: NVIDIA Corporation GP104 High Definition Audio Controller [10de:10f0] (rev a1)
IOMMU group 8
        00:1c.3 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #4 [8086:a293] (rev f0)
IOMMU group 6
        00:17.0 SATA controller [0106]: Intel Corporation 200 Series PCH SATA controller [AHCI mode] [8086:a282]
IOMMU group 4
        00:14.0 USB controller [0c03]: Intel Corporation 200 Series PCH USB 3.0 xHCI Controller [8086:a2af]
        00:14.2 Signal processing controller [1180]: Intel Corporation 200 Series PCH Thermal Subsystem [8086:a2b1]
IOMMU group 12
        04:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd Device [144d:a808]
IOMMU group 2
        00:02.0 VGA compatible controller [0300]: Intel Corporation Device [8086:3e92]
IOMMU group 10
        00:1f.0 ISA bridge [0601]: Intel Corporation Device [8086:a2c9]
        00:1f.2 Memory controller [0580]: Intel Corporation 200 Series PCH PMC [8086:a2a1]
        00:1f.3 Audio device [0403]: Intel Corporation 200 Series PCH HD Audio [8086:a2f0]
        00:1f.4 SMBus [0c05]: Intel Corporation 200 Series PCH SMBus Controller [8086:a2a3]
IOMMU group 0
        00:00.0 Host bridge [0600]: Intel Corporation Device [8086:3ec2] (rev 07)
IOMMU group 9
        00:1d.0 PCI bridge [0604]: Intel Corporation 200 Series PCH PCI Express Root Port #9 [8086:a298] (rev f0)
```

My IOMMU group 1 is perfect, which is basically just my nvidia GPU and its HD
audio device. The PCI bridge can be ignored but it needs to be removed from the
pcieport driver before we can bind the gpu to the vm.

Though if you have things like your network card inside the same group then
grabbing it is going to be more difficult. You can try an ACS patch which may be
able to break down your IOMMU groups further. I don't need it with this setup.
Yet.


# X server stuff

Like I mentioned before the host graphics is equally important and I do more
work on the host. I use a generic `xorg.conf` that is generated by
`nvidia-xconfig` to configure my graphics initially.

I leave that file untouched in `/etc/X11`.

I have a seperate Xorg configuration that I use for the Intel i915

```
Section "Device"
        Identifier  "intel"
        Driver      "modesetting"
        BusID       "PCI:0:2:0"
EndSection
```

This filed is named `intel.xorg.conf` and dumped in the same directory.

Update:

I no longer patch `xinit`. I started using a login manager: SLiM, which also
starts a consolekit session (makes things easier). I keep the default config as
`/etc/slim.conf.orig` and symlink either that or `/etc/slim.conf.intel` which
has the server argument `-config xorg.conf.intel` to `/etc/slim.conf`.

# Relevant scripts

Here's how I start and stop my VM and bind to different environments. This is
not perfect (neither is anything in this post) and criticism is welcome. Pull
requests too.

I don't use virt-manager or libvirt because

* I don't want another layer of abstraction
* XML makes life difficult and there are still parameters that can't be passed
in via the configuration file
* XML
* Did I mention XML?

My scripts are `sh` compatible and do not require any special shell.

This part prepares the drivers and binding to vfio-pci as required. I don't
think that the rebinding of vtconsole is required.

```sh
#!/bin/sh

# PCIe device ids
GPU=0000:01:00.0
GPU_SND=0000:01:00.1
PCI_BRIDGE=0000:00:01.0

# this script is intended to be run as root
# @TODO put in a check against whoami or $USER

function perror() {
  echo "$@" 1>&2;
}

function bus_rescan() {
  perror "[*] rescaning pci bus"
  echo 1 > /sys/bus/pci/rescan
  sleep 1
}

# usage
# vfio-bind $GPU vfio-bind
function driver_bind() {
  DEV="$1"
  DRV="$2"
  VENDOR=$(< /sys/bus/pci/devices/$DEV/vendor)
  DEVICE=$(< /sys/bus/pci/devices/$DEV/device)
  if [ -e /sys/bus/pci/devices/$DEV/driver ]; then
    perror "[*] unbinding $DEV"
    echo $DEV > /sys/bus/pci/devices/$DEV/driver/unbind
    sleep 1
  else
    perror "[!] existing driver for $DEV not found"
  fi
  perror "[*] binding $DEV to $DRV"
  echo $VENDOR $DEVICE > /sys/bus/pci/drivers/$DRV/new_id
}

function remove_pci_bridge() {
  perror "[*] removing pci bridge"
  echo 1 > /sys/bus/pci/devices/$PCI_BRIDGE/remove
  bus_rescan
}

function unbind_fb_vtconsole() {
  perror "[*] removing efifb and vtconsole binding"
  perror "[!!] you WILL lose console"
  echo "efi-framebuffer.0" > /sys/bus/platform/drivers/efi-framebuffer/unbind
  sleep .5
  echo 0 > /sys/class/vtconsole/vtcon0/bind
  echo 0 > /sys/class/vtconsole/vtcon1/bind
}

function rebind_fb_vtconsole() {
  echo "efi-framebuffer.0" > /sys/bus/platform/drivers/efi-framebuffer/bind
  sleep .5
  echo 1 > /sys/class/vtconsole/vtcon0/bind
  echo 1 > /sys/class/vtconsole/vtcon1/bind
}
```

And then for the X server

```sh
killall X
sleep 5

#unbind_fb_vtconsole

driver_bind $GPU vfio-pci
driver_bind $GPU_SND vfio-pci
remove_pci_bridge

perror "[*] switching monitor 1 to mDP"
ddccontrol -r 0x60 -w 16 dev:/dev/i2c-3

touch /tmp/vm.lock
su alex -c startx

rebind_fb_vtconsole
```

And for when I'm done with the VM:

```sh
killall X
sleep 5

driver_bind $GPU nvidia
driver_bind $GPU_SND snd_hda_intel
bus_rescan

rm /tmp/vm.lock
ddccontrol -r 0x60 -w 15 dev:/dev/i2c-3
#rebind_fb_vtconsole
su alex -c startx
```

# QEMU parameters

I'm using Qemu 3.1.0 at the time of updating this article.


```
export QEMU_AUDIO_DRV=pa
qemu-system-x86_64 -enable-kvm -m 8G \
  -machine pc-q35-3.0,accel=kvm \
  -cpu host,kvm=off,hv_relaxed,hv_spinlocks=0x1fff,hv_time,hv_vapic,hv_vendor_id=0xDEADBEEFFF \
  -rtc clock=host,base=localtime \
  -smp 4,sockets=1,cores=2,threads=2 \
  -vga none \
  -soundhw ac97 \
  -device vfio-pci,host=01:00.0,multifunction=on,x-vga=on \
  -device vfio-pci,host=01:00.1 \
  -drive if=pflash,format=raw,file=/usr/share/edk2-ovmf/OVMF_CODE.fd \
  -device ide-cd,bus=ide.1,drive=virtiocd1 \
  -drive media=cdrom,file=/tank/vm/win10.iso,id=virtiocd1,if=none \
  -object input-linux,id=kbd,evdev=/dev/input/by-id/usb-04d9_USB-HID_Keyboard-event-kbd,grab_all=on,repeat=on \
  -object input-linux,id=mouse,evdev=/dev/input/by-id/usb-Logitech_G102_Prodigy_Gaming_Mouse_017F36743435-event-mouse \
  -object input-linux,id=kbd2,evdev=/dev/input/by-id/usb-Logitech_G102_Prodigy_Gaming_Mouse_017F36743435-if01-event-kbd,grab_all=on,repeat=on \
  -device virtio-scsi-pci,id=scsi0 \
  -device scsi-hd,bus=scsi0.0,drive=rootfs \
  -drive id=rootfs,file=/tank/vm/photog.qcow2,id=disk,format=qcow2,if=none \
```
