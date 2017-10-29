---
title: iOS OpenVPN Connect Error - read length inconsistency
date: 2017-10-29 19:46:31
tags: ios, openvpn
---

If you've had the same error as what's described 
[here](https://forums.openvpn.net/viewtopic.php?t=18197), then it just means
that you have to insert the given key files inline into the configuration.

The error usually gets cut off as well in portrait mode so it looks like this:

```
Profile error : read length inconsistency: /var/mobile..
```

Usually with Tunnelblick you just bundle the `ca.crt`, `client.key` and
`client.crt` into a folder and rename it as `server_name.tblk` but that will not
work for OpenVPN connect.

Instead do this

```
#ca ca.crt
<ca>
-----BEGIN CERTIFICATE-----
.
.
</ca>
```

And so on.


