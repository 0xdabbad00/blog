---
layout: post
title: Kaspersky video on new(?) white-listing concept
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1282094658'
  _edit_last: '2'
---
Kaspersky's Lab "SecureList" released a <a href="http://www.securelist.com/en/blog/291/Whitelisting_how_it_protects_us">video</a> last week discussing whitelisting as a "newer protection technology".  Considering <a href="http://en.wikipedia.org/wiki/Open_Source_Tripwire">Tripwire</a> was started in 1992, and I imagine other solutions, at least home-made, existed before then, I wouldn't say it's new.  How they're using it, and the problems they encounter might be new though.

Applying white-listing to a system you never update or introduce new software to can be fairly easy by simply hashing all the executables on the system, and periodically rehashing them at a later date to see if they've changed.  The biggest problem with this, is if you have been compromised, how can you trust your ability to hash the files?  Your application for performing the hash could have been compromised or could have been hooked in such a way as to hide the attacker's changes.  So the recommendation is to load the hard-drive into a trusted system and perform the hashing there.  An alternative solution, and likely better, depending on your requirements, would be to just re-image the drive, or utilize virtualization and revert to a snapshot periodically.  This would save you from attacks that may hide outside of the executables you're hashing, because you can't hash everything on the disk, because some things inevitably will change (I'll discuss this idea later).

I assume the "new" concept of white-listing is about applying it in systems where executables are updated and changed, such as home user systems.  In this environment, patches and new software are going to be introduced frequently, and white-listing would be applied here by maintaining a master list of allowed binaries.  <a href="http://www.bit9.com/">Bit9</a> can be leveraged here, but what about staying on the bleeding edge as new executables come out.  What do you do on patch Tuesday or some other auto-update?  Do you trust the new .exes?  Do you apply the patch that might be untrusted, or do you wait and go without the patch until the security company ok's it?

What about software from smaller vendors?  How does the security company obtains copies of that software?  The small company doesn't want to send it's software to a couple dozen security companies that aren't paying for it, so should they send hashes?  Or does the security company need something more to create it's white-lists?  Maybe it wants to know what functions the software calls, so it can try to stop exploits as they occur.  There is no standard format for this, and no standard means of sending this to all the security companies, and no benefit for the vendor.

White-listing does need to be used more, but how can it be done effectively?
