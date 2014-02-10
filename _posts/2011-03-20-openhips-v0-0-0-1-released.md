---
layout: post
title: OpenHIPS v0.0.0.1 Released!
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1301860207'
  _edit_last: '2'
---
For the past month in my spare time I've been working on a project I've been calling OpenHIPS.  It's an open-source Host-Intrusion Prevention System (HIPS) for Windows.  In this initial release, it is just a user-friendly way of using <a href="http://blog.didierstevens.com/programs/heaplocker/">HeapLocker</a> by Didier Stevens, with it's most useful feature being the ability to detect when Adobe Acrobat has used lots of memory, which may be indicative of a heapspray, and kill it.

The goal of a HIPS is to be able to prevent exploits from working, and should be used in addition to keeping up with patches and <a href="http://www.microsoft.com/downloads/en/details.aspx?FamilyID=c6f0a6ee-05ac-4eb6-acd0-362559fd2f04">EMET</a>, which enforces ALSR, DEP, and other features.  My goal in OpenHIPS is to add mitigation techniques that are currently only theoretical or not user-friendly to use, and provide a platform for others to add their capabilities to.  It's focus will be on Windows 7, 32-bit and 64-bit systems, home user systems.

A lot of the work for this project is new to me, so please use at your own risk (preferably just try it out on a VM).  I'm still learning and still working on the project, and will provide further updates to this blog with how the project works and why I've made it the way it is.  There are so many features that still need to be added, that it's not even worth trying to record all the current "issues" (like it only protects 32-bit processes currently, you can't change the settings through the GUI, more settings are needed, etc.)

Once installed, it will show a little shield icon an "O" (for OpenHIPS) on the task bar, which when clicked will show the configuration settings it currently has set, which are the default settings used by HeapLocker, except I took out the ability to scan memory for strings, because I want to use <a href="http://code.google.com/p/yara-project/">YARA</a> instead so I can scan for multiple patterns.  OpenHIPS currently will only protect new processes, and not any processes currently running when you install, so you should restart Adobe Reader, Internet Explorer, Firefox, and Foxit if you have them open already.  You can test OpenHIPS is working by downloading and opening the file <a href="https://www.assembla.com/code/openhips/subversion/nodes/Releases/v0.0.0.1/spray-js.pdf?_format=raw&amp;rev=94">spray-js.pdf</a>.  This pdf will allocate 600MB of memory (it does not have any shell-code).  The threshold for OpenHIPS is 500MB before it asks you if you want to terminate the process.  In `./trunk/test/` are the python scripts used to create that file, which is based on the work from <a href="http://feliam.wordpress.com/2010/02/15/filling-adobes-heap/">http://feliam.wordpress.com/2010/02/15/filling-adobes-heap/</a>

<b>OpenHIPS v0.0.0.1</b>

<img src="http://0xdabbad00.com/wp-content/uploads/2011/03/openhips_logo.png" alt="OpenHIPS logo" /> <a href="https://www.assembla.com/wiki/show/openhips/">Source code</a> | <a href="https://www.assembla.com/code/openhips/subversion/nodes/Releases/v0.0.0.1/openhips-setup-v0.0.0.1.msi?_format=raw&amp;rev=94">Installer</a> | Test file <a href="https://www.assembla.com/code/openhips/subversion/nodes/Releases/v0.0.0.1/spray-js.pdf?_format=raw&amp;rev=94">spray-js.pdf</a>
