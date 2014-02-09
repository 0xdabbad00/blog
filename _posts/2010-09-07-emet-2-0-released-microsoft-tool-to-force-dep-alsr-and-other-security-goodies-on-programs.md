---
layout: post
title: ! 'EMET 2.0 released: Microsoft tool to force DEP, ALSR, and other security
  goodies on programs'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1283909836'
  _edit_last: '2'
---
Windows has some great security features to prevent exploits (DEP and ASLR), but they aren't always turned on.  Executables have to be compiled in such a way that they request these features, or you as the user have to force all executables on your system to use these features (which isn't a clicky-clicky task).  

<a href="http://blogs.technet.com/b/srd/archive/2010/09/02/enhanced-mitigation-experience-toolkit-emet-v2-0-0.aspx">EMET</a> is a free tool from Microsoft released last week that gives you clicky access to turn this stuff on all over.  Also it adds some other anti-exploit tricks to prevent shell-code from getting run or prevent it from doing what it was written to do.  You can also force these features be enabled on a per processor basis (ASLR for example, can't be turned on everywhere because it potentially can cause problems apparently).  

This is a cool tool because you can make your box more secure very easily without installing a full HIPS product that would slow your box down.  Cool things it also adds are Structure Exception Handler Overwrite Protection (SEHOP), some heap spray mitigation (by pre-alloc'ing the addresses that Metasploit uses apparently in it's exploits (according to the BlueHat video about this)), and Export Address Table Access Filtering (EAF) (I don't really get how this works, but the User Guide hints that it's using debug registers for read protections).  Ok, these might impact your system a little, but so far I'm not seeing a difference like I do with HIPS's.
