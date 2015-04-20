---
layout: post
title: Application White-Listing Expectations
categories: []
tags: []
status: publish
type: post
date: 2015-04-16 14:00:00 -06:00
published: true
---

**Summary**: Lower your expectations.

In infosec, one definition of an exploit is something that takes advantage of a flaw.  To be a "flaw", it must be unexpected behavior, and to "take advantage", it must result in greater privileges.

Many discussions of exploits, bypasses, or attacks on white-listing systems show what they would claim is unexpected behavior because I believe many **people's expectations of application white-listing are too high**.  For example, there are many posts about using powershell to bypass white-listing solutions.  How did they get the powershell script there in the first place and give it execution?  Many of these posts can be summarized as "Given arbitrary code execution, I was able to get arbitrary code execution."  If you obtained no new privileges, is this an exploit? Was any of this unexpected?  Perhaps you gained an easier persistence mechaninism, so that's a new privilege, but after the point where you have arbitrary code execution, you've probably passed the point where application white-listing should be considered useful.



What threats does a protection stop?
------------------------------------
Consider a firewall using white-listing for connections it allows through.  It has a single rule to only allow connections on port 80.  You have an HTTP server and a database server, and don't want external access to the database server.  Let's say you're running IIS and someone get's RCE on your system with that new http.sys vuln (CVE-2015-1635).  Has the firewall been bypassed?  Let's say they install an old-school backdoor that connects out to an IRC server.  Now has the firewall been bypassed? Was any of this an unexpected "flaw" of the firewall?   Now let's say the attacker installs a backdoor that listens on port 666.  The firewall has ended up blocking this.  Was that it's purpose though?  It's nice the firewall did this, but was blocking backdoors a threat you were relying on the firewall to stop?

If you run [grsecurity](https://grsecurity.net/) on a server, and someone guesses your ssh password, has grsecurity been bypassed?  If you run EMET and you get phished, has EMET been bypassed?  That sounds ridiculous if you know what those protections are for, because you don't expect them to do anything under these circumstances.  Your expectations have not been violated. No flaw or "exploit" of those products has been used.  But to the non-technical executive who thinks his systems are super secure because they have one of these mitigations, and finds out it's been compromised, he might assume that the product is flawed.  That's the problem we have today with application white-listing.  There are differences in the assumption of threats application white-listing is supposed to stop.  

A recent example was the post [How to bypass Google’s Santa LOCKDOWN mode](https://reverse.put.as/2015/04/13/how-to-bypass-googles-santa-lockdown-mode/).  It discusses using `DYLD_INSERT_LIBRARIES` to inject their library.  Clearly this requires arbitrary code execution.  It also mentions using something like DLL hijacking on OS X, which again in most cases requires you to somehow get code execution on the box.  The authors that describe those techniques believe they have found unexpected behavior in the application white-listing solution, whereas for me personally, I don't feel this is really a concern to those solutions.  

I respect both of those gentlemen and so what we have is a difference in opinions on what a white-listing solution should do.  Many people share their opinion, and so their work is important because it shows others where application white-listing does not provide protections that some assume it does, or believe it should.

What threats does application white-listing stop?
-------------------------------------------------
So what do I think the value of application white-listing is?  I wrote about [this](http://0xdabbad00.com/2013/01/22/value-of-white-listing/) back at the start of 2013.  Expanding on that, here are some threats application white-listing protects against:

- Tricking the user into downloading and executing a file.
- Using autoruns (conficker) or .lnk vuln (stuxnet) from USBs, and some other command injection type tricks.
- Some MiTM attacks. For example, the threats from [the-backdoor-factory](https://github.com/secretsquirrel/the-backdoor-factory) of downloading a file over HTTP that has been modified in transit.  
- Running out of date or unpatched software.
- Users running software your company doesn't have licenses for (not a security threat so much as a compliance and legal issue).

These are the concerns application white-listing deals with.  If it helps stop others, that's great, but if you rely on it for more than this, I think you're going to have a bad time.
