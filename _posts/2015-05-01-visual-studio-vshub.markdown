---
layout:     post-no-pic
title:      "How to fix: HttpHostx64.exe has stopped working"
subtitle:   "Fixing the 'Microsoft.VsHub.Server.HttpHostx64.exe has stopped working' error for Visual Studio"
date:       2015-05-01 13:18:00
author:     "Thomas Barthelemy"
tags:       [windows, visualstudio]
---

# The HttpHostx64.exe error

In Visual Studio CTP and Visual Studio RC I started having the following error
repeatedly:

    Microsoft.VsHub.Server.HttpHostx64.exe has stopped working

The online reference to hit are pretty limited and I could not find a solution
online so I decided to mess up a little with those file configurations and ended
up finding a working solution:

The crashing file is located in the following folder:

    C:\Program Files (x86)\Common Files\Microsoft Shared\VsHub\1.0.0.0

# The fix

It seems that the x86 version of the file does not have the same issue, so
a simple fix, until a better one is officially published by Microsoft:

- Stop Visual Studio if started
- rename `Microsoft.VsHub.Server.HttpHostx64.exe` to `Microsoft.VsHub.Server.HttpHostx64.exe.bak`
- Create a copy of `Microsoft.VsHub.Server.HttpHost.exe` and rename it `Microsoft.VsHub.Server.HttpHostx64.exe`
- Re-start Visual Studio

The issue has been reported and a thread is available here:

[https://connect.microsoft.com/VisualStudio/feedback/details/1293295/httphostx64-exe-has-stopped-working](https://connect.microsoft.com/VisualStudio/feedback/details/1293295/httphostx64-exe-has-stopped-working)
