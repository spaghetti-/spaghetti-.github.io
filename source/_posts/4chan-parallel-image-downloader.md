---
title: Simple shell function to download images from 4chan threads in parallel
date: 2017-01-16 05:17:51
author: alex
tags: shell, scripting
---

This is an old function that I wrote  a while back to download content from 4chan threads. It stopped working recently since they changed the link format. Fixed it up because I was bored.

It uses `xargs` to dispatch downloads to threads and uses only commonly used unix tools. If we replace the `\d` regex with `[0-9]` it'll work with plain `grep` without relying on PCRE.

```sh
function 4get() {
  curl -k -s $1 | egrep -o \
    "\/\/is\d?\.4chan\.org\/gif\/\d+\.(webm|gif|jpeg|jpg|png)" | \
    sed 's/^/https:/; s/is\d?\.4chan\.org/i\.4cdn\.org/' | uniq | \
    xargs -n 1 -P 12 curl -# -O
}
```

The script can also be found on [gist](https://gist.github.com/spaghetti-/bd4fcbc6cd3a9b55b02940e78d24fdeb).
