---
layout: post
title: Windows Package Manager needed to improve secruity
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1318125001'
  _edit_last: '2'
---
In order to keep a computer system secure you should:
<ul>
<li>Obtain software from trusted sources.
<li>Keep that software up-to-date.
<li>Have additional lines of defense in case vulnerabilities exist in that software, or to catch any mistakes you might make.
</ul>

Assuming that the hardware you use is trusted, and that software exists which can be trusted, I propose a service is needed which would allow you to more easily obtain that trusted software and keep it up-to-date. Two currently existing solutions are <a href="ninite.com">ninite.com</a> and <a href="allmyapps.com">allmyapps.com</a>.  These work by allowing you to download a single executable which will install a variety of common applications (chrome, 7zip, winmerge, openoffice, etc.) without any additional user interaction and keep these up-to-date.  I would like to see a more open solution that allows for a “profile” of the applications I personally would want to install when I get a new system/VM, refresh a system, etc.  I’d login and it would generate a download for me with all my favorite apps.  Additionally, as I install apps on a system, it would keep track of this for my profile.  Ideally, I’d be doing that installation anyway through this package manager.

I want this to be a more open solution so that I can have more fine-grained control over how things are installed.  For example, when I have to set-up a new system for myself, I download eclipse, putty, and SysInternals tools and put them in a new dir C:\bin.  I also want more fine-grained control to allow extensions to applications to be installed.  For example, when I install Firefox or Chrome, the first thing I do is install Adblock Plus and Ghostery.

I want all of these to be kept up-to-date through a single auto-updater.  No company focuses on their auto-updater.  They are important, but there is not a strong business case to have a good one, and a developer that works on some great open-source tool is not also going to focus on a great auto-updater for it.  They won’t make sure it is using the best secure download solution, that minimizes band-width usage, installs apps at the most ideal time, and provides users with a lot of options about how it should work.  A single auto-updater would aggregate all the annoying pop-ups I normally get into one app.  It would remove the need for multiple processes to all be running at the same time just so they can check once/day if my software is up-to-date.  Most importantly it could have more smarts than current auto-updaters.  It could realize that I am shutting down and apply updates then, like Windows.  It could realize I don’t have Adobe reader up, and am away from the computer, and apply an update then.

With such a package manager, you could select commonly used profiles people have generated for public consumption.  They could also aggregate partial-profiles.  For example, one partial-profile might install Firefox and extensions for it, whereas another would have Chrome and extensions for it, so they could choose one or the other or both.  Another partial-security profile would have <a href="http://support.microsoft.com/kb/2458544">EMET</a> and an anti-virus product.  It would have a partial-profile for web developers, graphic designers, etc.

This package manager help solve the problem of trusted downloads and keeping software up-to-date along with providing some of the additional lines of defenses.  This could also set white-listing rules to ensure that only executables downloaded and installed through the package manager are allowed to execute in the system.

Today there are more malware than goodware.  Therefore, a greater effort should be made to ensure the only software running on a system is goodware, instead of trying to identify and remove malware.  For business systems that use proprietary software, the openness of this package manager would allow for the custom apps to be easily white-listed for that network.  I'm not sure how exactly this should work, but this should exist, and I believe it would greatly improve the security of Windows computers.
