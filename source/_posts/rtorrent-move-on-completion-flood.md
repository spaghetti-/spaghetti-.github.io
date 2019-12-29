---
title: rtorrent + flood - Moving completed files out to a different directory
date: 2019-12-28 16:24:10
tags:
  - rtorrent
  - flood
  - bittorrent
  - zfs
---

ZFS is known to have performance overheads caused by the 16KB read/write of
random blocks by bittorrent[0]. One way of avoiding this fragmentation is by
moving files from a dataset that is made specifically for staging to another
that can hold the files better for sequential I/O (for example, downloading them
off the server).

The Arch wiki has a section on this[1] that works for vanilla rtorrent, I
assume. I manage rtorrent by using flood[2], an excellent web based UI. It does
not use the watch directories AFAIK so this part

```
schedule2 = watch_directory_1,10,10,"load.start=/home/user/torrents/watch/*.torrent,d.custom1.set=/home/user/torrents/complete"
```

is not useful anymore. Instead we can just define the path directly

```
method.insert = d.move_to_complete, simple, "d.directory.set=$argument.1=; execute=mkdir,-p,$argument.1=; execute=mv,-u,$argument.0=,$argument.1=; d.save_full_session="
method.set_key = event.download.finished,move_complete,"d.move_to_complete=$d.data_path=,/your/path/here/"
```

If you are running rtorrent in a container (as I am, an alpine container
particularly) then `mv -u` may not work if you're using the binary provided by
busybox instead of the one from coreutils that you're familiar with. Change it
up

```
method.insert = d.move_to_complete, simple, "d.directory.set=$argument.1=; execute=mkdir,-p,$argument.1=; execute=mv,$argument.0=,$argument.1=; d.save_full_session="
method.set_key = event.download.finished,move_complete,"d.move_to_complete=$d.data_path=,/your/path/here/"
```


[0] http://open-zfs.org/wiki/Performance_tuning#Bit_Torrent

[1] https://wiki.archlinux.org/index.php/RTorrent#Manage_completed_files

[2] https://github.com/Flood-UI/flood
