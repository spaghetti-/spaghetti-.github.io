---
title: Parallel rsync
date: 2018-07-19 18:21:24
tags:
---

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
