---
title: Automatically turning on a lamp when in a video meeting
date: 2020-07-31 14:15:04
tags:
  - automation
  - macos
---

I've been working from home for a long time now (ever since COVID-19) and one of
the complaints I've had is that I sit facing away from a window (monitors and
webcam facing the window). This means my colleagues get a really dark view of my
face after auto brightness accounts for the window or my ceiling lights.

To fix this, I bought a cheap IKEA TERTIAL lamp[0] which retails for 20 bucks,
and aimed it at the wall in front of me to diffuse the light. I used a spare
Phillips Hue bulb that I had lying around giving me easy controls from Apple
Home.

![bulb](https://i.imgur.com/3ka27KB.jpg)

That's still too many actions for my liking because I'd have to open up
control center, go to Home, and turn on the light. Not to mention doing the same
thing one my meeting is over.

It's pretty straightforward to automate this, we need to

* detect if the webcam is enabled
* (in my case) setup a proxy over another computer to avoid storing creds on my
  corp laptop
* control the light over the Hue bridge using its API

Detecting if the webcam is enabled is surprisingly easy. All the links on Google
point to checking for processing using `AppleCamera` or `VDC` but I don't think
that works anymore (at least, it does not work for me on Catalina). Instead,
we'll use the `Console.app` program or the command line equivalent `log` that
comes built-in to MacOS.

Detecting webcam use
--------------------

```sh
sudo log show --predicate 'subsystem == "com.apple.VDCAssistant" && \
    eventMessage CONTAINS[c] "kCameraStream"' --last 1m --no-debug --no-pager \
    --style compact | tail -n +2)
```

which tells us if events like `kCameraStreamStart` and `kCameraStreamStop`
occur, which seems to occur each time the camera starts and stops. Note that
this command does not seem to work as a normal, unpriviledged user and instead,
root access. There's probably a way to work around this, but this is a hack
*cough* short term solution *cough*.

Controlling lights over the Hue bridge with it's API is relatively
straightforward, we just need to get a token from the hue developer portal. Once
you have the token you can then control the light with a simple `PUT` query like
so:

```sh
#!/bin/zsh

TOKEN=your_token_here
BULB_ID=7

read M
if [ "$M" = "bulb:on" ]; then
  curl -X PUT -H "Content-Type: application/json" -d '{"on": true}' \
    http://${HUE_HUB_IP}/api/${TOKEN}/lights/${BULB_ID}/state
elif [ "$M" = "bulb:off" ]; then
  curl -X PUT -H "Content-Type: application/json" -d '{"on": false}' \
    http://${HUE_HUB_IP}/api/${TOKEN}/lights/${BULB_ID}/state
fi
echo
```

I have made the script read for events like `bulb:on` from stdin that'll be then
used to toggle the state.

Proxying the command
--------------------

I keep this running on a personal computer in my household and expose it over a
TCP port with socat. It's kept daemonized with start-stop-daemon and OpenRC.

```sh
#!/sbin/openrc-run

depend() {
        need net
}

start() {
        ebegin "Starting bulb control"
        start-stop-daemon -S -m -p ${MEET_PIDFILE} -u alex --quiet --background \
          socat -- -u tcp-l:6338,reuseaddr,fork system:/home/alex/.homebridge/meet.sh
        eend $?
}

stop() {
        ebegin "Stopping bulb control"
        kill -- -`cat ${MEET_PIDFILE}`

        rm ${MEET_PIDFILE}
        eend $?
}
```

Each time there's anything sent to tcp:6338 it runs the script which then
controls the bulb. With this, all the corp laptop sees is a string sent to a
local computer using netcat.

Tying it all together
---------------------

The script to detect the webcam state is put in the root users crontab on the
laptop. It then sends the appropriate string to the proxy script to toggle the
physical state of the bulb.

I set the script to check for the status of the webcam every 1 minute, which is
a relatively cheap operation and does not affect the performance of the laptop.
Just in case, I also verified that cron commands aren't triggered when the
laptop is sleeping[1].


The user experience is pretty good, I hop on a call with Google
Meet, which enables the camera, which triggers the lamp, and which automatically
turns it off after I'm done.



Ref
---
[0] https://www.ikea.com/sg/en/p/tertial-work-lamp-dark-grey-50355395/

[1] https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/ScheduledJobs.html#//apple_ref/doc/uid/10000172i-CH1-SW3
