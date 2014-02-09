---
layout: post
title: Restricted information on stuxnet
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _edit_lock: '1285546544'
---
I hate sounding paranoid or like a conspiracy theorist, but the delays regarding information about Stuxnet is unsettling.  First, we have the delays in how long it has taken to disclose some of it's 0-day exploits as I discussed in my post <a href="http://0xdabbad00.com/2010/09/16/what-kind-of-disclosure-is-this/">What kind of disclosure is this?</a>.  But now it's come out that Stuxnet didn't just have the ability to access SCADA systems using a default Siemen's password, but according to <a href="http://www.symantec.com/connect/blogs/stuxnet-introduces-first-known-rootkit-scada-devices">Symantec's post</a> it "contains 70 encrypted code blocks" and these are used for "hooking enumeration, read, and write functions".  It doesn't seem that anyone has figured out what these code blocks are really doing (yeah they are hiding themselves, but they have to be doing something there to bother hiding them).  The best source so far has been a post from a previously unknown German company called Langer that posted <a href="http://www.langner.com/en/index.htm">"Stuxnet is a directed attack -- 'hack of the century'"</a> on Sep 13, 2010.  There they make note that Stuxnet contain fingerprints to check for a very specific target.  <a href="http://frank.geekheim.de/?p=1189">Other sources</a> on the net suspect that Stuxnet was designed specifically to go after Iran's Bushehr or Natanz centrifuges.

Ok, so it takes a while to reverse engineer code, but from it's Juny 17 discovery to Langer's post on Sep 13, is a very long time for there to have been no discussion of these additional SCADA abilities.  I would assume every a/v company and other researchers immediately jumped at looking at Stuxnet when it was found and the initial information about it was disclosed.  Now I could understand that if this was written by the Israeli's as <a href="http://www.digitalbond.com/index.php/2010/09/16/stuxnet-target-theory/">some suspect</a>, then CheckPoint (maker's of Zone Alarm and an Israeli company) might not post anything, or if it was Russians behind it than Kaspersky might not post anything, but there are security companies in pretty much every country (and also I can't assume that Russia or Israel or any other nation would be able to control what it's security companies do).   It is to their advantage to provide info about malware because it serves as free advertising for their expertise.

My take-away from all this has been that security companies are all shady.  You could argue that no security company really has any incentive to figure out what all malware does, that they just need to know that Stuxnet is bad, and know how to identify it and remove it, but my counter argument is then that none of these a/v companies would detect all of Stuxnet then.  They'd detect the infection mechanism, but not the code that messes with the SCADA stuff (which seems to run on Windows).  So you'd think you're SCADA stuff is all safe now, while all the while Stuxnet's real purpose is still running.  So these are the options I see: security companies are shady, the don't do a good job of reverse engineering, or I'm incorrect that they have any incentives to post about their findings.
