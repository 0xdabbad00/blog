---
layout: post
title: Prevx patching Windows
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1291499666'
  _edit_last: '2'
---
So Microsoft is doing a pretty bad job lately at patching known, and currently exploited, vulnerabilities, so Prevx is giving themselves some kudos for providing, as part of their Anti-malware software, <a href="http://www.prevx.com/blog/162/
Windows-day-exploit-QA-session.html">a way to stop one of these exploits</a>.

As far as I know, this is not common for one company to be offering patches for another companies software.  This is what I advocated back in one of my <a href="http://0xdabbad00.com/2010/09/26/lets-start-making-solutions/">older posts</a>, but it just doesn't happen. Microsoft sometimes "patches" old software to get it to run on new versions of their OS's (as part of their application compatibility stuff) but these are usually bug fixes (so for example, they'll patch some old unsupported game so when it allocates an array, it allocates more memory than is actually asking for, because the original code had a buffer overflow in it that would crash the program when you tried to run it on the new OS).  Also, Microsoft, as part of their <a href="http://0xdabbad00.com/2010/09/12/how-emet-works/">EMET tool</a> (for adding DEP and ASLR to any program) uses this same application compatibility technology to "patch" programs.  But they don't specifically block one vulnerability.  I guess some PSPs do perform some patching (McAfee ePolicy Orchestrator comes to mind), but I guess no one ever flaunts this like Prevx is doing. 
