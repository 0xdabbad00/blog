---
layout: post
title: Sandboxing
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1291501281'
  _edit_last: '2'
---
Adobe announced <a href="http://blogs.adobe.com/asset/
2010/11/adobe-reader-x-is-here.html">Reader X</a>, which is a sandboxed version of their product.  Congrats to them, because they suck at security, as has been concluded from the endless onslaught of exploits being thrown at them this year.  BUT, is a sandbox the answer?  It sounds really cool in theory, a sandbox is a play area where an app can do whatever it wants and not affect anything else... in theory.  In reality, what IE, Microsoft Office, Google Chrome, and now Reader (all sandboxed products) have done is leverage Windows integrity levels, which is a feature that came out with Windows Vista (if you still rock XP, you're out of luck, and also you need to upgrade your 9 year-old OS cheap-ass!).  So integrity levels enforce the concept of least privilege, so that a process has the fewest number of privileges it needs to work.  This is done by spawning a new process from the medium integrity level process to the low integrity level process, and then communicating with it.  Check out this <a href="http://blog.didierstevens.com/2010/11/19/quickpost-adobe-reader-x/">pic</a> to see what is done, and some links if you want to know more.

So anyway, this is like a half-ass version of Linux's chroot jail, because for example, any low integrity process can still mess with any other low integrity process. Now, low integrity processes really are limited in where they can write in the file system and registry, so you've broken tons of malware, BUT this does not stop you from getting remote code execution with your original exploits for Adobe in any way shape or form.  If you can get execution and you know of a privilege escalation exploit in Windows, chances are you can break out of this "sandbox" and go about your business.  So yeah, Adobe raised the bar, but it's not as cool as you might initially think.  It's not a VM, or emulated execution or something that might remove those exploits in the first place, it's just another hoop to jump through such as compiling the programs with DEP and ASLR (or using Microsoft's EMET tool to force these poorly written products to use those technologies, seriously getting the Windows project settings right when you compile an exe is way harder than it should be).
