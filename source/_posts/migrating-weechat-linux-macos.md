---
title: Migrating weechat configuration from Linux to MacOS (and vice versa)
date: 2018-07-25 14:29:13
tags: weechat
---

If you are using SSL, the weechat will refuse to connect because the location of
the ca file is different on MacOS and Linux (`weechat.network.gnutls_ca_file`).

On MacOS you can set (for homebrew users)

```
/set weechat.network.gnutls_ca_file "/usr/local/etc/openssl/cert.pem"
```

And on Linux

```
/set weechat.network.gnutls_ca_file "/etc/ssl/certs/ca-certificates.crt"
```

And you should be good to go.
