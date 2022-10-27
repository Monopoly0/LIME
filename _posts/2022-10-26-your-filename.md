---
published: false
---
---
title: Writing a New Post
author: cotes
date: 2022-10-26 14:10:00 +0800
categories: [POC, Mac]
tags: [mac, priv-esc]
render_with_liquid: false
---
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c2/OS_X_security_layers.svg/1200px-OS_X_security_layers.svg.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />

## What's System Integrity Protection (SIP) and Why Do We Care?

According to Apple, "System Integrity Protection is a security technology in OS X El Capitan and later that's designed to help prevent potentially malicious software from modifying protected files and folders on your Mac. System Integrity Protection restricts the root user account and limits the actions that the root user can perform on protected parts of the Mac operating system.

Before System Integrity Protection, the root user had no permission restrictions, so it could access any system folder or app on your Mac. Software obtained root-level access when you entered your administrator name and password to install the software. That allowed the software to modify or overwrite any system file or app."

**Apple also specefies what parts of system System Integrity Protection protects which are:**

- `/System`
- `/usr`
- `/bin`
- `/sbin`
- `/var`
- Apps that are pre-installed with OS X

**Protected Paths and apps that third-party apps and installers can continue to write to include:**

- `/Applications`
- `/Library`
- `/usr/local`

Also, "System Integrity Protection is designed to allow modification of these protected parts only by processes that are signed by Apple and have special entitlements to write to system files, such as Apple software updates and Apple installers. Apps that you download from the Mac App Store already work with System Integrity Protection. Other third-party software, if it conflicts with System Integrity Protection, might be set aside when you upgrade to OS X El Capitan or later." It also helps prevent software from selecting a startup disk but thats not important right now for our current goals.

;
## SIP and Notarized Code

With the introduction of notarized code Apple has pretty much made it standard to visit the security prefrences everytime they want to run un-notarized code. Any piece of software that doesn't have a valid code signature and isn't submitted via Apple's Xcode to be notarized will read as this:

<img src="https://media.idownloadblog.com/wp-content/uploads/2017/04/remove-open-program-dialog-before.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
     
Which Mac users are conditioned to ignore (Maybe if Apple replaced this with a Virus Total Scan attached users would actually pay attention to this). Apple doesn't like code on your system that doesn't go through the notarization proccess so they won't protect it with SIP. This also includes privelaged applications that are un-notarized. 

## Exploiting SIP Misconfigurations

I initially discovered this exploit while messing with cheat engine and attacking web assembely. Attaching a debugger to Proccess's that were protected by SIP (including applications that existed outside of SIP's protected directories) was blocked by the system.

<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/SIP%20Circumvet/Screen%20Shot%202022-08-19%20at%205.22.12%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />

Now if we do the same thing to a non-notarized application we can observe how SIP ignores protection measures regardless of privelages. For demo I'll run libreWolf from the application folder (a un-notarized piece of software) and try to attach my debugger to it.

<img src="https://github.com/LimeIncOfficial/Blog-Repo/blob/main/SIP%20Circumvet/Screen%20Shot%202022-08-19%20at%205.39.03%20AM.png?raw=true"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
## Potential 

Malicious Agents with a foothold in the system as user could privelage escalate using SIP ignored non-notarized software as a gateway into root. Obviously this relies on a foothold on the system to be present. Apple could mitigate this with a simple update to allow for SIP protection to extend to non-notarized software as well. For now you can stop this midway by installing BlockBlock (https://objective-see.org/products/blockblock.html) an open source tool by Objective-See to see whenever un-notarized code tries to launch and have the ability to block it before it runs.


