---
layout: post
title: SlopFinder
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1352640455'
  _edit_last: '2'
---
I was reading <a href="http://www.amazon.com/Bug-Hunters-Diary-Software-Security/dp/1593273851">A Bug Hunter's Diary</a> by Tobias Klein last week and was reminded of an idea I had <a href="http://0xdabbad00.com/2010/09/12/idea-tool-to-rate-use-of-defense-in-depth/">long ago</a> to check if a software application is using things like DEP and ASLR.  There are lots of tools to do this (in the book it uses <a href="http://technet.microsoft.com/en-us/sysinternals/bb896653.aspx">Process Explorer</a> and <a href="http://www.erratasec.com/lookingglass.html">LookingGlass</a>), but you have to download and run those locally. This is 2012, and downloading apps is old-school.  I made a simple web app to use HTML5 hotness to allow users to drag and drop entire directories and analyze them client side.  You can drag and drop your entire "Program Files" dir!

Checking if an executable has DEP or ASLR set is really easy.  All you need to do is parse the PE header to get to IMAGE_OPTIONAL_HEADER.DllCharacteristics and check the bits in that WORD to see what features are turned on.

Admittedly, SlopFinder is sort of sloppy javascript code.  This is like when you correct someone's grammar or spelling and make mistakes yourself.  SlopFinder does not work on 64-bit binaries, has almost no error handling or user feedback when things go wrong, and only checks DEP and ASLR.

Here is a screenshot of my Foxit reader directory:
<a href="http://0xdabbad00.com/wp-content/uploads/2012/11/slopfinder2.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/11/slopfinder2-150x150.png" alt="" title="SlopFinder on FoxIt reader" width="150" height="150" class="alignnone size-thumbnail wp-image-523" /></a>

Try <a href="http://icebuddha.com/slopfinder.htm">SlopFinder</a> out now.
