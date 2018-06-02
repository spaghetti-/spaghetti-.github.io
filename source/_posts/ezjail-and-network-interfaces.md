---
title: ezjail and Network Interfaces
date: 2018-06-02 17:31:06
tags: 
  - ezjail
  - freebsd
  - networking
---

I was updating and adding a new service to one of my servers when I needed to
fiddle with ezjail. ezjail, as its name suggests, makes creating and maintaing
jails quite simple. According to the [handbook](https://www.freebsd.org/doc/handbook/jails-ezjail.html) we can create jails by doing

```sh
ezjail-admin create dnsjail 'lo1|127.0.1.1,em0|192.168.1.50'
```

After changing the ip to match your needs obviously, but it fails to mention
that if you have a problem with the second part `em0`, which is often your
external interface, `ezjail-admin restart jail` will tear down your default
route and all connectivity to the interface. `ezjail` adds aliases to it and I
am guessing, on tear down it also downs the primary ip.

I've been meaning to add a warning to the handbook when I get some time.

To prevent mishaps from happening in the future I reccomend onestarting ezjail
when making changes. If youve set (as you do) `ezjail_enable="YES"` in rc.conf,
the problem with the external interface will persist through reboots. If you
don't have serial console setup on the server you are out of luck without
rescuing via netboot.

As it turns out, online.net's FreeBSD rescue images that they provide for rescue
don't boot either and you're forced to go with ubuntu which does not come with a
write enabled UFS driver by default. The kernel also compiles by default
disallowing force loading kernel modules with vermagic mismatches.

To get around it, I had to first get the source for the running kernel by doing

```sh
apt-get source linux-image-`uname -r`
```
and then by copying over the `Makefile` from `/usr/src/linux-*` to your own
build machine and then compiling the single module with our changes (enabling
write). The module was then copied back to the server and loaded in, which
allowed us to mound the root as readwrite. rc.conf was then modified and
rebooted.

More information on compiling singular kernel modules can be found here.
