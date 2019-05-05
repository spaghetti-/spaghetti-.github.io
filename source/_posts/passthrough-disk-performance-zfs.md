---
title: Tuning VM disk performance for a passthrough setup (host on zfs)
date: 2019-05-05 16:06:49
tags:
  - zfs
  - vm
  - vfio
  - passthrough
---

The entirety of this post is thanks to /u/mercenary_sysadmin on
reddit [(post 1)](https://www.reddit.com/r/zfs/comments/8sbq0y/seeking_advice_for_zfs_layout_for_mixed/e0y8hay/) [(post 2, with benchmarks)](https://www.reddit.com/r/zfs/comments/4oa4xb/how_do_you_configure_virtmanager_to_use_zfs_zvol/d4bofw9/).
Plenty of beers on me if you see this and are ever in Singapore.

Cliffnotes version for me to remember:

### Gaming VM

```
zfs set recordsize=16K
qemu-img create -f qcow2 games.qcow2 500G -o cluster_size=16K
# zfs set compression=lz4 # already inherited
zfs set atime=off tank/vm
```

Here's a benchmark from CrystalDiskMark with the above created image:


![](https://i.imgur.com/rIR0qbn.png)


Disk is attached to the VM as:

```
-device virtio-scsi-pci,id=scsi0 \
-device scsi-hd,bus=scsi0.0,drive=rootfs \
-drive id=rootfs,file=/tank/vm/games.qcow2,id=disk,format=qcow2,if=none
```

### Photography

```
zfs set recordsize=1M
zfs set atime=off tank/photos
zfs set xattr=sa tank/photos
```
