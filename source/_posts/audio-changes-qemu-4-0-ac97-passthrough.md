---
title: Changes in qemu 4.0 for passthrough (particularly for pulseaudio + ac97)
date: 2019-05-05 17:02:08
tags:
  - vm
  - audio
  - passthrough
---

## Version

```
QEMU emulator version 4.0.0
```

```
Installed versions:  4.0.0(05:38:52 PM 05/04/2019)(aio alsa bzip2 caps curl fdt filecaps gtk iscsi jpeg ncurses nls opengl pin-upstream-blobs png pulseaudio sdl seccomp usb vhost-net vnc xattr -accessibi
lity -capstone -debug -glusterfs -gnutls -infiniband -lzo -nfs -numa -python -rbd -sasl -selinux -smartcard -snappy -spice -ssh -static -static-user -systemtap -tci -test -usbredir -vde -virgl -virtfs -vte -x
en -xfs KERNEL="linux -FreeBSD" PYTHON_TARGETS="python2_7 python3_6 -python3_5 -python3_7" QEMU_SOFTMMU_TARGETS="x86_64 -aarch64 -alpha -arm -cris -hppa -i386 -lm32 -m68k -microblaze -microblazeel -mips -mips
64 -mips64el -mipsel -moxie -nios2 -or1k -ppc -ppc64 -riscv32 -riscv64 -s390x -sh4 -sh4eb -sparc -sparc64 -tricore -unicore32 -xtensa -xtensaeb" QEMU_USER_TARGETS="-aarch64 -aarch64_be -alpha -arm -armeb -cri
s -hppa -i386 -m68k -microblaze -microblazeel -mips -mips64 -mips64el -mipsel -mipsn32 -mipsn32el -nios2 -or1k -ppc -ppc64 -ppc64abi32 -ppc64le -riscv32 -riscv64 -s390x -sh4 -sh4eb -sparc -sparc32plus -sparc6
4 -tilegx -x86_64 -xtensa -xtensaeb")
```
## Machine Type

To integrate a few audio patches I upgraded the machine type from 3.0 to 4.0.
Turns out when you upgrade the machine type you also need `kernel_irqchip=on` or
the nvidia driver wont work.

```
-  -machine pc-q35-3.0,accel=kvm \
+  -machine pc-q35-4.0,kernel_irqchip=on,accel=kvm \
```

## Audio dev

With the previous patch I had to do

```
export QEMU_AUDIO_DRV=pa
```

which will now cause qemu to abort immediately on launch with `-soundhw ac97`.
Instead we should do

```
+  -audiodev pa,id=pa1,server=`pactl info | grep 'Server String' | awk '{print $3}'`
```

Note how I've to get the pulse socket dynamically from `pactl` on my
systemd free Gentoo installation. If it's on a systemd machine it can be
replaced with `server=/run/user/1000/pulse/native`.

Thanks to /u/spheenik [(reddit announcement)](https://www.reddit.com/r/VFIO/comments/b1crpi/qemu_40_due_soon_might_bring_superb_audio_test_now/) whose audio patches I've been using
for a long time.
