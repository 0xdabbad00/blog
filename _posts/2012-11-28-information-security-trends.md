---
layout: post
title: Information Security Trends
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1358192087'
  _edit_last: '2'
---
This post is my opinion of what the important trends going on lately are, based on talks at cons from this past Summer.

<h3>Asymmetric Defense</h3>
Asymmetric defense is doing something that is relatively easy/inexpensive for the defender that makes it very difficult/costly for the attacker.  Example, memory protections like DEP and ASLR for a developer are very easy to turn on, and now life becomes very hard for an exploit developer.  My tool <a href="http://icebuddha.com/slopfinder.htm">SlopFinder</a> is a quick way to ensure these basic protections are enabled.  There are other compiler and OS protections like this (SEHOP, Null pointer protection).  Windows 8 and <a href="http://blogs.technet.com/b/srd/archive/2012/05/15/introducing-emet-v3.aspx">EMET v3</a> have advanced those.  Microsoft also had their <a href="http://blogs.technet.com/b/srd/archive/2012/07/26/technical-analysis-of-the-top-bluehat-prize-submissions.aspx">BlueHat challenge for ideas to protect against ROP exploits</a>.

<b>Historically, the attacker only needed to "win once"</b> and now people are trying to make it so defenders only need to "win once".  Many things can go wrong in exploitation, so defenders need to increase the likelihood things will go wrong for shellcode.  As an example, <a href="http://ambuships.com/">AmbushIPS</a> by <a href="http://www.scriptjunkie.us/">scriptjunkie</a> was released at the BlackHat BSides (there are a growing numbers of conferences within conferences now, such as the Skytalks at Defcon).  AmbushIPS is a platform for allowing people to define rules for functions to hook and block processes from doing certain things.  

Piotr Bania's project <a href="http://www.piotrbania.com/all/protty/">Protty</a> (discussed in <a href="http://www.phrack.org/issues.html?issue=63&id=15#article">Phrack 63</a> back in 2005) is another such example, which tried to block shellcode from reading the import table, but unfortunately it didn't work well (too slow), but such an idea and others like it, may have a revival. Likewise, my <a href="http://0xdabbad00.com/project-openhips/">OpenHIPS</a> sought to make it possible for people to run <a href="http://code.google.com/p/yara-project/">YARA</a> signatures on the memory of processes, but I ultimately thought my idea would not work out well.  Maybe a combo of AmbushIPS and OpenHIPS in order to leverage YARA signatures might be useful?

The general concept here is:
<ol>
<li>Write secure code so an attacker can't get control of EIP (better compilers, languages,...),
<li>but if they do, make it so they can't execute their shellcode (DEP, ASLR, EMET...)
<li>but if they do, make it so their shellcode can't do anything useful (AmbushIPS, Protty,...), 
<li>but if it can have freedom, make it run in a sandbox, etc.
</ol>

<h3>Sandboxes</h3>
There are various talks at cons about getting through so-and-so's sandbox.  Beyond just sandboxes, there are VM's.  People try to do different tasks in different browsers (one browser that has Java installed for banking possibly).  Joanna Rutkowska has her <a href="http://qubes-os.org/trac">QubesOS</a> to make each process run inside it's own VM, and you can have multiple and separate browsers that can't communicate.  I think this is an important concept.  We are going to see more focus on attackers taking maximum advantage of what they are able to get.  Everything is in the browser now, so if you want to read someone's email, steal their bank creds, etc., all the data you want to steal is inside the browser.  This means:
<ul>
<li>You don't need ring-0: The browser runs in userland.
<li>You don't need admin/root privs: The browser runs as the user.
<li>You don't need to get outside the sandbox: All the data you want is already in your process.
</ul>

This breaks down as we look at things like Google Chrome which run different windows in different processes, but there may be more ways to <b>jump between</b> sandboxed processes than outside of it.

<h3>Reducing the attack surface area</h3>
Windows 8 actually seems to be getting pretty locked down.  There are tons of protections for memory exploits all over it now, so even if you write sloppy C code, it's hard to get controlled execution of shellcode.  EMET helps back-port these memory protections to older OS's.  Much of code for Windows is now .NET which is more secure than C (harder to shoot yourself in the foot with memory vulnerabilities at least).  The <b>App Store for Windows should allow white-listing</b> to be possible and secure the downloading of code you want to execute.

I think the next step here will be to <b>reduce the attack surface area</b>.  Java has issues now, so remove the midi support and other things from Java you don't need for whatever you use java for.  There have been font exploits, so remove unneeded fonts from the system.  Stuxnet and Agent.BTZ exposed issues with what Explorer does when a USB drive is inserted, so figure out ways to remove all that extra unneeded functionality.

<h3>Porting security</h3>
Many talks nowadays are just on porting known concepts to new architectures.  Owning phones is no different than owning anything else, just there is more white-listing, fewer memory protections, and greater diversity in versions of what is installed.  We'll likely see "advances" in these other architectures for the next few years of porting concepts from Windows (DEP, ASLR, etc.) to phones, SCADA equipment, whatever, until it just becomes common practice that for any new OS or device/OS/architecture, the first questions asked are if it employs DEP, ASLR, etc. in the same way that now for any communications mechanism there are those who want to know if it is encrypted.

<h3>Trust</h3>
More talks are about how someone got an app that can do naughty things onto someone's app store.  People will be focusing more on how to avoid this, and I think the only real way is determining trust.  "App store" is just a different way of saying "white listing", and how do you white list?  You trust something that is open-source, you have a trusted entity review the code of the thing (Microsoft driver certification), you ensure you can identify the creator behind the code, other methods?  Remove the anonymity and a lot of crimes start to go away.

<h3>Privacy</h3>
Privacy concerns and trust issues are becoming more prevalent.  Sites tracking people and colluding with their info (something I know a fair bit about from my past work), and devices (the apps on them) phoning home and having access to more data than they should.  I think we will soon see a <b>good "rootkit"</b> for a phone that will hook calls to things like GetAddressBook() and GetGPSLocation() (or whatever the actual function call is) and the rootkit will return fake data to your pandora app and everything else except those processes that actually should have access (like your email client and phone).  We may see attacks on data miners to flood them with fake data, or more research into where data goes as its sold and resold between marketers.

As for the network privacy concerns, I would expect to see greater use of things like TOR and VPN's, not just for privacy, but also for security, because if you're at a place like Defcon, you want your wifi to run through a VPN to avoid MiTM.  You're going to want all your sites to use SSL somehow.  Network IDS's are already dead, so you may see <b>host based NIDS's</b> that can alert on data after it's been decrypted on the host, or provide some sort of proxied experience.
