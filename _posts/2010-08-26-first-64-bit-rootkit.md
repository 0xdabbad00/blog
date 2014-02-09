---
layout: post
title: First 64-bit rootkit!
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1283909627'
  _edit_last: '2'
---
Prevx has posted about finding a new version of TDL3 in the wild that works on 64-bit Windows (<a href="http://www.prevx.com/blog/154/TDL-rootkit-x-goes-in-the-wild.html">http://www.prevx.com/blog/154/TDL-rootkit-x-goes-in-the-wild.html</a>).  They don't give much for details, other than saying it patches the MBR in order to get execution early and has to reboot the machine, these are both required to get past the driver verification and PatchGuard.  I'll be following this closely to find out what TDL3 is actually doing. Expect more posts as I find out more info.
