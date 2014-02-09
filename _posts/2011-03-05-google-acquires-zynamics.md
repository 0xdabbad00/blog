---
layout: post
title: Google acquires Zynamics
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1299335778'
  _edit_last: '2'
---
A few days ago on 2011.03.01 Google bought <a href="zynamics.com">Zynamics</a>.  I've held off on posting this just for the sake of posting it, because I wanted to find someone who would say why.  Zynamics makes software to compare disassembles of code (to compare versions of malware, identify vulns that patches were made for, look for copyright infringement by finding people using your code, etc.), dissect pdf's for malware, and similar work.

I guess one reason is simply to add more security people (as they did in the past with Travis Ormandy).  Google has been pushing hard to keep Chrome secure, as is evidenced by their <a href="https://threatpost.com/en_us/blogs/google-fixes-19-bugs-chrome-pays-14k-bug-bounties-030111">payouts of bounties for security bugs</a>.  Granted, they only paid out $14K for 19 bugs in that post, and it is assumed that a good security bug can be worth a lot more than that average price of roughly $1K/bug.

Additionally, Google has an interest of identifying sites hosting malicious content and flagging emails with malicious attachments, malicious pdf's you view through it's reader, or docs you look at it through it's various document tools.

With Google's massive computing resources to draw on, and smart statisticians, they might do more in the classification and analysis realm, and maybe release their own A/V software, which they might have an interest in for their Android phones, Chrome OS, and would just give them greater presence in general.

However, this deal might not have been great for everyone at Zynamics, because just a few days later on 2011.03.4, one of the components used in Zynamics tools, Hexer, was <a href="http://www.the-interweb.com/serendipity/index.php?/archives/131-Release-of-JHexView-1.0.html">released as open-source</a>.  Interestingly, that post says that Zynamic's tools are all Java, which is odd, since all Zynamic's tools deal with things at the byte level, and Java is the worst language for doing anything at that level.
