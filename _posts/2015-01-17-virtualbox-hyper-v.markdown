---
layout:     post-no-pic
title:      "VT-X Not available in VirtualBox"
subtitle:   "A love triangle between VT-x, Hyper-V and VirtualBox"
date:       2015-01-17 10:14:00
author:     "Thomas Barthelemy"
---

I found myself recently in an interesting situation, which all started with a simple need: I need to run a 64-bit virtual machine on my laptop.

# VirtualBox

I fired up my VirtualBox and quickly realised that it only offered me 32-bit options to create a new Virtual Machine.
This usually means that the VT-x option(Intel Hardware Virtualization) must be disabled in my BIOS, although I was pretty sure it was enabled already.
I restarted my laptop, checked the BIOS, and as I remembered: VT-x was indeed enabled. I tried to switch it off and on again with a restart in-between but
still no 64-bit options.

# Diagnostic

I downloaded the Intel Processor Identification Utility, which can tell whether the VT-x is enabled or not, and unsurprisingly it told me it wasn't.
Back to the BIOS, disabled VT-x, reboot, start the tool: VT-x enabled!
Wow that's weird, but if it works... Well not quite, VirtualBox was still not getting it.

I went a bit further and updated BIOS and related drivers, just in case, with still no effect, I was starting to think my laptop good cursed by a powerful wizard or something big was hidden in plain sight.
I took some time to read a little about Hardware virtualization and realized that it does not play well with other, and Windows 8 (or above) is providing a native hardware virtualization tool: Hyper-V.

# The light

Hyper-V isn't something that comes enabled by default but I installed the Windows Phone SDK not so long ago which sets it up automatically.
A little trip under: Program and Features -> Features to disabled Hyper-V, and I finally broke the curse.

What happens if you need them both?
If you need both Hyper-V and VT-x at the same time, on the same OS, sorry it's just not how it works.
What you can do is have VT-x enabled and run a Virtual Machine with Hyper-V, which is obviously a bit resource-heavy.

Another possibility is to create a way to quickly switch between those two (as enabling-disabling from the features menu is definitely not quick),
One way to achieve that is to create a new boot entry using **bcdedit** with the **hypervisorlaunchtype off** parameter:

{% highlight Windows batch files %}
C:\>bcdedit /copy {current} /d "No Hyper-V" 
The entry was successfully copied to {ff-23-113-824e-5c5144ea}. 

C:\>bcdedit /set {ff-23-113-824e-5c5144ea} hypervisorlaunchtype off 
The operation completed successfully.
{% endhighlight %}
