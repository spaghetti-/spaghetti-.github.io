---
title: Parallel rsync
date: 2018-07-19 18:21:24
tags:
---

Note: This post is outdated. I don't saturate my link at home because of
bandwidth delay product[0]. I knew this at the point of writing the post, but I
could never get my calculated theoritical speed (even considering bdp). It was
most probably because I did not consider the number of hops between me and the
server when doing the calculations.

I removed the extra steps to parallelize rsync like so
```sh
ssh root@bifrost.wg find -L /var/media/dir \
      -type f -printf '%P\\0' | \
      parallel -0 --no-run-if-empty rsync -avrRsz --progress \
      root@bifrost.wg:"/var/media/dir/{}" /tank/tv/dir
```

[0] https://en.wikipedia.org/wiki/Bandwidth-delay_product

Original post:

Not sure if its because of my ISP packet shaping or otherwise, but single
connections never saturate my 1gbps link at home. I needed to move a lot of
music files off a server back to a new storage pool so:

* first make a list of files
```sh
find /zroot/DATA/music/ -type f -name "*.flac" \
       -exec dirname "{}" \; | uniq
```

* split it into 12 parts
```
split -l N list
```

* use xargs to parallel rsync
```sh
find . -name "xa*" -print0 | xargs -0 -I{} -n 1 -P12 rsync -avzr \
       --no-links --quiet --files-from={} alex@host:/ /tank/music/
```

This gets me on average 25MB/s as opposed to 2MB/s on single threaded rsync.
