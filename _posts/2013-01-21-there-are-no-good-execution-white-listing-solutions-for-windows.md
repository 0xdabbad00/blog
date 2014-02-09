---
layout: post
title: There are no good execution white-listing solutions for Windows
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1358798246'
  _edit_last: '2'
---
In a recent F-Secure blog post <a href="http://www.f-secure.com/weblog/archives/00002487.html">Protecting Against Attacks Similar to "Red October"</a>, @<a href="https://twitter.com/jarnomn">jarnomn</a> advocates a couple of ideas that could have protected those that fell prey to Red October.  One of them is white-listing which files are allowed to execute.  This is often raised as one of the solutions that could have protected those that fell prey to <a href="https://en.wikipedia.org/wiki/Operation_Aurora">Aurora</a> and almost every other use of malware when these operations hit the news.  F-Secure has advocated white-listing in the past: After Flame, Mikko Hypponen wrote a popular post in Wired titled <a href="http://www.wired.com/threatlevel/2012/06/internet-security-fail/">Why Antivirus Companies Like Mine Failed to Catch Flame and Stuxnet</a>.

I am in full support of white-listing.  My <a href="http://0xdabbad00.com/2010/08/17/kaspersky-video-on-new-white-listing-concept/">first post</a> on this blog back in 2010 was about white-listing.  But there are not currently any good solutions for implementing it.  @jarnomn advocates using the built-in Windows technology AppLocker.  Unfortunately, AppLocker only exists in the following Windows editions:
<ul>
<li>Windows Server 2008 R2
<li>Windows 7 Ultimate and Enterprise
<li>Windows Server 2012
<li>Windows 8 Enterprise
</ul>

All of the malware attacks that have spawned discussion about white-listing have been on user systems, so we can ignore the server OS's.  So that leaves just Enterprise editions of Windows 7 and 8, plus Ultimate on 7.  It's also odd that AppLocker works on fewer editions of Windows 8 than Windows 7, which leads me to believe that Microsoft might feel they made a mistake with it and are trying to kill it off?  In any case, because it does not work on the Pro or Home editions, this is not an option to most users and even many businesses.

Microsoft's AppLocker is considered a successor to Microsoft's Software Restriction Policies (SRP).  SRP first showed up with Windows XP, and the best guide to using it is probably the NSA's <a href="https://www.nsa.gov/ia/_files/os/win2k/Application_Whitelisting_Using_SRP.pdf">Application Whitelisting Using Software Restriction Policies</a>.  You should also skim through Microsoft's technet article <a href="http://technet.microsoft.com/en-us/library/bb457006.aspx">Using Software Restriction Policies to Protect Against Unauthorized Software</a>.

When you first go to use SRP, it sets up policies to only allow applications within the "Program Files" or "Windows" directories to run.  So their "white-listing" is initially just based on path's, and it is not very restrictive: Malware can run out of C:\Windows\Temp\ still, which is a likely place for malware to run from anyway.  Using paths for policies can be effective, and is what the NSA's paper advocated, but it's not a very strong approach.

The two most important white-listing capabilities of SRP are to use hashes or certificates.  Hashes for SRP will be difficult to maintain though, as there is no easy import/export functionality.  So any time you wish to update a system using SRP hashes, you're going to have a lot of work.  Creating rules for certificates requires you to actually have a copy of the certificate, so I can't figure out how to just white-list anything signed by Microsoft.  I should be able to just say "Here is an example of a file signed by Microsoft, make sure any other file matches that same signer."  This is what AppLocker allows you to do. The other major problem with SRP is there is no audit capability to allow you to create a bunch of test rules and have the results of what would be blocked logged.  This means that you need a test system to try out your rules on, or else you could easily accidentally brick your box with overly restrictive rules.

Concluding, <b>neither AppLocker nor Software Restriction Policies are good solutions</b>.

Other free solutions for white-listing on Windows are simply scripts to check if any files changed since the last run of the script.  These include 
<ul>
<li><a href="http://www.ossec.net/doc/manual/syscheck/index.html">Syscheck</a>, which is part of <a href="http://www.ossec.net/doc/index.html">OSSEC</a> which is part of <a href="http://communities.alienvault.com/">OSSIM</a> from <a href="http://www.alienvault.com/">AlienVault</a>
<li><a href="http://www.tripwire.com/">Tripwire</a>
</ul>

These just let you know how recently you got owned (assuming the malware doesn't modify the checking software), instead of actually protecting you.

What I want is an application for Windows 7 Pro that when I install it, it assumes my system is clean, and automatically detects and white-lists all executables (.exe and .dll), ideally based on who they are signed by, and possibly warns me if there are any weird signatures (I don't know what this might mean), and just hashes anything else.  Then it will be able to warn me when a new executable appears or tries to execute.  <a href="https://www.bit9.com/products/bit9-parity-suite.php">Bit9 Parity Suite</a> may be a solution to this, but I don't think they sell to consumers or small businesses.

Final thoughts: Lately I've writing about unmet security needs in the consumer and small business market, and really even the big business market.  This trend will continue for a while, and I apologize that there is no great conclusion to these posts to tell you what to do.   These blog posts are forcing me to research these problems and try to identify existing solutions out there that maybe I'm not seeing.  Eventually, I hope this motivates me to get angry enough to fix all these.
