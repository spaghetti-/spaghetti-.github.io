---
title: WhatsApp chat backup getting stuck/no progress
date: 2017-10-29 19:57:45
tags: ios
---

Sometimes the iOS WhatsApp application refuses to perform its chat backup
properly. Yesterday, my backup was stuck at 152kB/couple of GB for the better
part of the day. Restarting your phone, restarting the app, toggling cellular
and airplane modes will not fix the issue.

Instead, head on over to `Settings > iCloud > iCloud Drive` and turn that off.
It'll warn that there is some data from WhatsApp that is not saved yet, but
you can turn it off anyway. No data was lost on my end.

Just turn it back on, head on over to WhatsApp and restart the backup process
manually. This fixes the issue.
