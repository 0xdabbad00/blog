---
layout: post
title: Value of white-listing
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1358886502'
  _edit_last: '2'
---
There has been some argument that white-listing does not provide much value to users.  

First, let's imagine for a moment that there are no more memory corruption vulnerabilities: In this imaginary world, a new version of EMET or something comes out and magically they all are gone.  Even without memory corruption exploits, there are still ways of getting hacked.  These include:
<ul>
<li>Tricking the user into downloading and executing a .exe
<li>Tricking the user into opening a file in a location under your control such that you can use DLL hijacking
<li>Using autoruns (conficker) or .lnk vuln (stuxnet) from USBs
<li>MiTM attacks.
  <ul>
  <li>You go to download putty.exe from it's actual site and someone gives you a trojaned version.
  <li>Your software goes to auto-update and doesn't check what it is updating with (such as <a href="http://www.h-online.com/security/news/item/German-spyware-exploits-iTunes-vulnerability-1382455.html">itunes once did</a>).
  </ul>
</ul>

White-listing what you allow to run on your system would be effective at stopping all of these.  This, however, assumes that the user does not disable our white-listing solution and just allow it to run, but well, you can only make children's scissors so dull: A dedicated boy can still cut himself.  I believe a white-listing solution could be devised that is both <b>performant</b> and  <b>accurate</b>.  By this I mean that it does not slow down your system and that when it blocks a file, it is blocking it for a good reason, and because this should happen rarely and with ferocity, it will inhibit a user from trying to work around this white-listing.

Let's move back into reality, and current exploitation scenarios.  I believe some of the Java exploits that have happened recently were not memory corruptions, and ultimately were used to write files out to disk and execute them.  Although they could execute arbitrary java code, java is still somewhat restricted in what it can do.  In any case, even if the exploitation can do whatever it wants, it is still a valid strategy to not be low hanging fruit and use every defense you have available.  Most exploits today are just getting arbitrary execution so they can drop a file and execute it.  This stops that.  

Those exploit payloads can be modified, but someone would need to modify it to get me.  If/when I get owned, I want to know I didn't get owned by the B-team using their weakest weapons.  If I get owned in part of something like Operation Aurora, I want to somehow know that the attackers had to use their more advanced techniques and more senior guys against me.  They had to spend a little more time and money to get me.  Even if I get owned, I want to keep my pride. ...To be honest though, if someone takes the time to own me in something like Operation Aurora, I'd be pretty proud: Directed attacks are directed at valued targets.

So if we can block whole classes of attacks, plus make a little pain for other classes of attacks, then why shouldn't we?  The main reason, is if it causes us more pain to apply the defense than it's worth, and today, as I pointed out in my last post <a href="http://0xdabbad00.com/2013/01/21/there-are-no-good-execution-white-listing-solutions-for-windows/">there are no good execution white-listing solutions for windows</a>.  So I'm going to have to prove you all wrong eventually that it can be done. :P
