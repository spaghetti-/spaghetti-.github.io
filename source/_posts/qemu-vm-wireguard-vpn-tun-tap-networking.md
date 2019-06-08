---
title: Setting up TUN/TAP networking for QEMU VM's (and bonus wireguard)
date: 2019-05-13 17:44:18
tags:
  - qemu
  - passthrough
  - vpn
  - wireguard
  - networking
---

I needed to setup a VM that could _only_ have connectivity through a
specific wireguard endpoint. Previoiusly, the VM was using user mode networking
(SLiRP) which required no setup. I asked around on IRC and was told that
libvirt takes care of that stuff for you so most folks don't care - but I
already have a few VM's and am not ready for black magic or XML. I will
eventually have to move them, I suppose. Windows VMs don't take kindly to even
the slightest of parameter changes, so if there's a tool that takes the
commandline flags and generates the XML, that'd be awesome.

If you really need SLiRP networking (user mode) to work and want to route
through wireguard, you could consider using network namespaces to achieve the
same result. I like the flexibility offered by the TAP setup that lets my host
and LAN devices interfact with the VM in a much more _normal_ way (subjective).

## Network Setup

### Diagram
```
     bridge masters

  +--------+   +--------+
  | vm0    |   |  vm1   |
  +---+----+   +---+----+
      |            |
+-----+------------+-----+   +-------------+   +----------+
|         br0            |   |     wg0     |   |   eth0   |
|     10.0.3.0/24        |   | 10.0.0.1/24 |   |          |
+------------------------+   +-------------+   +----------+
```

`vm0` and `vm1` are TUN devices.

Caveat: Not all traffic in the host is routed through the wireguard interface so
we need some `ip -4 rule` usage to route.

### Wireguard

This is the most straightforward so let's get it out of the way. You need the
kernel module loaded.

```sh
# modprobe wireguard
ip link add dev wg0 type wireguard
ip addr add 10.0.0.2/24 dev wg0
wg set wg0 private-key
wg set wg0 peer PUBLIC_KEY_HERE allowed-ips 0.0.0.0/0 endpoint stty.io:port
ip link set up dev wg0
```

And on the "server" do the same, but with some modifications

```sh
wg set wg0 peer PUBLIC_KEY_HERE allowed-ips 10.0.0.2/32, 10.0.3.0/24
iptables -A FORWARD -i wg0 -j ACCEPT
iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

You'd also need to enable IP forwarding

```sh
sudo sysctl -w net.ipv4.ip_forward=1
```

Notice how we've allowed the bridge's ip range in allowed-ips. Bridge setup
time:


### Bridge and TUN devices

If you need to run qemu without requiring root user, setup a group `netdev` and
add yourself to it. Logout and login to take effect. `id` must report that your
user is in `netdev` before you proceed. Also create a new udev rule

```
KERNEL=="tun", GROUP="netdev", MODE="0660", OPTIONS+="static_node=net/tun"
```

to configure your TUN devices to use the group. I put mine in
`/etc/udev/rules.d/10-qemu.rules` along with some other stuff for passthrough.

Your kernel needs "Universal TUN/TAP devices" enabled.

For the bridge:
```sh
ip link add br0 type bridge
ip addr add 10.0.3.0/24 dev br0
ip link set up dev br0
```

For a DHCP server we can use `dnsmasq`

```sh
sudo dnsmasq --interface=br0 --bind-interfaces --dhcp-range=10.0.3.0,10.0.3.255
```

For the VM's adapters

```sh
ip tuntap add vm0 mode tap group netdev
ip link set vm0 up
ip link set vm0 master br0
```

### QEMU Parameters

```sh
-device virtio-net-pci,netdev=vm0 \
-netdev tap,id=vm0,ifname=vm0,script=no \
```

Important points

* instead of `virtio-net-pci` you can use anything else like `e1000` which has
inbuilt drivers in windows. I installed it manually from the driver disk
provided by Red Hat.
* `script=no` is important, else QEMU tries to init a new adapter itself and
that required superuser
* if `ifname=vm0` is not provided `script=` doesn't work because qemu doesn't
know which adapter to configure and it'll try to run anyway.

You'll get an error saying qemu can't configure the device which looks
suspiciously like you didn't setup your permissions correctly, but it's not.


At this point your VM should boot and the network adapter should recieve an IP
address from dnsmasq.

### Routing and Internet Connectivity

Like I mentioned before, the host does not have a default route through the
wireguard endpoint. We only want traffic from the bridge, or even, specific VMs
to go through the endpoint.

```sh
ip route add default via 10.0.0.1 dev wg0 priority 1 table 1
ip rule add from 10.0.3.0/24 priority 1 lookup 1
```

This should be sufficient to do policy based routing from the VM to the internet
and to the host. You should verify this by pinging from within the VM and from
the host.

## Thanks to

Documentation found on the [kvm website](https://www.linux-kvm.org/page/Networking)
and IRC users grawity, tds and others.
