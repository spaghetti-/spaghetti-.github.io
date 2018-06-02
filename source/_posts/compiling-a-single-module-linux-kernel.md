---
title: Compiling a single linux kernel module with matching vermagic
date: 2018-06-02 17:46:46
tags: 
  - linux
  - kernel
---

I recently got locked out of a server because of configuration mistake and had
to rescue a FreeBSD server by netbooting Ubuntu (only image provided by the host
that booted). My root was running on UFS and as expected Ubuntu does not come
with a write enabled UFS kernel module.

I set up a virtual build box on my local machine as it was faster than setting
everything up on the netboot'd image (plus since it was totally in memory, if
anything went wrong it'd be best to setup tooling offline). But just in case we
need to remember, the dependencies are 

```
libncurses-dev
libssl
build-essentials
bc
```
After booting into the Ubuntu 16.04 image, we can then fetch the kernel source 
by doing

```sh
apt-get source linux-image-`uname -r`
```

This would install `/usr/src/linux-headers-version` (replace version) with the
kernel source. Then we need to scp the `Makefile` found in the directory to the
same directory on our build box.

We can then enable `UFS_FS_WRITE=y` in the kernel config. Then call

```
make oldconfig
make prepare
make modules_prepare
make M=fs/ufs/ modules
```

Then the resulting `ufs.ko` can be scp'd back to the server to be loaded. The
vermagic will match if the same `Makefile` was used across both kernel
compilations.
