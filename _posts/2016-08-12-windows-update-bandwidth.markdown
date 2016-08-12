---
layout:     post-no-pic
title:      "How To: Stop Windows windows update from using all your bandwidth"
subtitle:   "When Windows Update won't even share enough to keep Skype connected"
date:       2016-08-12 00:00:00
author:     "Thomas Barthelemy"
tags:       [windows]
---

What's Up with Windows Update
=============================

I first noticed that something was wrong in one of the latest 
Windows 10 Preview before it's release:

I could open any web page, my Skype was desperately trying to connect
and my wife was asking me what's happening with our internet.
 
It was fairly quick to notice my computer was capping the download
bandwidth of our connection, and this through the `svchost` process
which told me a lot but not enough.

Windows was downloading updates (sometime without even showing it
in the update panel) and keeping all bandwidth for himself.

<img alt="A little bit of internet"
     src="http://i.imgur.com/e2C3w.jpg"
     style="max-width: 80%;"
>

Obviously this is a non-problem when you have a 500Mb connection but
from the depth of Cambodia, just waiting for the update to finish could
mean a few hours. That's a nope.

P2P Updates
===========
Part of the new features introduced in windows 10 is the ability
to get updates through P2P (local or not) to improve download speed.
Although I understand and approve the general idea,
we are trying to reduce the speed, so you can turn it off in:

`Windows Update -> Advanced Options -> Choose how updates are delivered`

and chose `PCs on my local network`, this will still allow you to get
update from other computer but only in your local network which won't
ruin your available bandwidth.

Although this improves things a bit, overall it did not fixes thing for
me... Let's keep searching then.

BITS
====

The updates happen through 2 main windows components: WUDO and BITS.

WUDO is the Windows Update Delivery Optimization is part of the
Windows Update for Business and is used for the P2P installation that
we disabled already.

The Background Intelligent Transfer Service (BITS) is commonly used by
Windows to download updates, can we do something about this little guy?

When something misbehave, it's time to put some rules up and as an
hereditary tribute of his Windows Server brother, Windows 7 and up
have a Group Policy editor that we can use to put some rules.

To open the Local Group Policy Editor from the command line:

 - Click Start , type `gpedit.msc` in the Start Search box, and then press ENTER.

To set a bandwidth rule on the BITS:

 - Navigate to `Administrative Templates` -> `Network` -> `Background Intelligent Transfer Service (BITS)`
 - Open `Limit the maximum network bandwidth for BITS background transfers`
 - Set it to `Enabled`
 - Set the time range and maximum transfer rate
 - OK
 
Problem solved!

3rd Party Solutions
===================

There are quite a few 3rd party tools to monitor and control bandwidth
usage, the most named is NetLimited that removed the control features
from the free tier of its application earlier this year (you can always
download older versions though).

That will work just fine, but I'd prefer not having something running
on my computer I don't totally trust and that is looking at my traffic.


Conclusion
==========

It's really interesting to see how many Windows tools are hidden
and how much control your are given through them.

Share in the comments the best hidden feature for you in Windows!