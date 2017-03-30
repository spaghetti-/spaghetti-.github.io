---
title: Bootstrap sane defaults for sshd and ipfw
date: 2017-03-09 17:34:54
tags: freebsd
author: alex
---

I often find myself spawning random DigitalOcean FreeBSD
droplets for prototyping web services. Just because they are temporary does not
mean that they should not have secure defaults however. Here's a short script
that sets up `sshd` as well as initializes the `ipfw` firewall.

For AWS instances we're better off using VPC with sane security groups to do our
firewalling for us.

It can be run using

```sh
ssh user@host 'sh -s' < script
```

It disables password authentication and allows only public key based
authentication, disables root login, limits retries. As for ipfw, it only allows
tcp in on port 22 by default. 

```sh
#!/bin/sh

head -1 ~/.ssh/authorized_keys | ssh-keygen -l -f - > /dev/null 2>&1

if [ $? != 0 ];
then
  echo "invalid public key found in authorized_keys, exiting.."
  exit
fi

cp /etc/ssh/sshd_config ~/sshd_config.bak

sudo tee /etc/ssh/sshd_config <<EOF
Port 22
SyslogFacility AUTH
LogLevel INFO
LoginGraceTime 2m
StrictModes yes
MaxAuthTries 6
MaxSessions 10
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2
PasswordAuthentication no
PermitEmptyPasswords no
Subsystem       sftp    /usr/libexec/sftp-server
PermitRootLogin no
EOF

sudo tee -a /etc/rc.conf <<EOF
firewall_enable="YES"
firewall_quiet="YES"
firewall_type="workstation"
firewall_myservices="22/tcp"
firewall_allowservices="any"
firewall_logdeny="YES"
EOF

sudo service sshd restart
sudo service ipfw start
```

A maintained version of this script can be found on
[gist](https://gist.github.com/spaghetti-/40896fb8f6cdc56851f894291d149ae5).

The blog syntax highlighter seems to remove newlines from the source.
