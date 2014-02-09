---
layout: post
title: Invisible Things Lab posts Linux priv escalation
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1282092863'
  _edit_last: '2'
---
The post at <a href="http://theinvisiblethings.blogspot.com/2010/08/skeletons-hidden-in-linux-closet.html">http://theinvisiblethings.blogspot.com/2010/08/skeletons-hidden-in-linux-closet.html</a> discusses an interesting priv escalation attack they found and is discussed in the <a href="http://www.invisiblethingslab.com/resources/misc-2010/xorg-large-memory-attacks.pdf">paper</a> linked off of the post.  I don't play around enough in *nix to understand exactly how it works, but it seems to be an interesting class of attack that looks very hard to patch and fix.

This is great marketing for <a href="http://qubes-os.org/">Qubes</a> (developed by the Invisible Things Lab as a virtualization solution).  The post from Joanna avoids marketing Qubes too much, but in the comments she makes this remark "In Qubes we're not *adding* an additional layer of abstraction -- we're *replacing* the buggy Linux monolithic kernel, with something orders of  magnitude less buggy".
