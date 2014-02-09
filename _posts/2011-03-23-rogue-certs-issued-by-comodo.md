---
layout: post
title: Rogue certs issued by Comodo
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1300935783'
  _edit_last: '2'
---
<a href="http://www.h-online.com/security/news/item/SSL-meltdown-forces-browser-developers-to-update-1213358.html">The H</a> and <a href="http://www.f-secure.com/weblog/archives/00002128.html">F-Secure</a> are reporting that rogue SSL certs were issued by Comodo.  These include certs for mail.google.com (GMail), login.live.com (Hotmail et al), www.google.com, login.yahoo.com, login.skype.com, and addons.mozilla.org.  This is really bad.  This means that anyone that can <a href="http://en.wikipedia.org/wiki/Man-in-the-middle_attack">MiTM</a> (such as joe hacker setting up a rogue access point, or a nation, as F-Secure thinks Iran is behind this, that can control it's ISP's) can intercept your login credentials to those sites, or provide a means to download firefox plugins using that addons.mozilla.org, and even the most paranoid and knowledgeable person won't be able to tell they aren't accessing the real site.

SSL is one of the cornerstones of e-commerce, so this is really bad.  I don't believe browsers do a great job of ensuring the sites I go to are trusted.  As is often the case, the protocol and whatnot is probably pretty secure, but the implementation is not.  Looking at my Firefox browser, I see roughly 70 companies are trusted at root certificate authorities, of which I can easily see a dozen that I don't want to trust, because I doubt any sites I go to actually use them (such as Turktrust and Hongkong Post) .  I need to investigate what options exist to better enforce trust.
