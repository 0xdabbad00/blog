---
layout: post
title: Fingerprinting malware using YARA
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1325652923'
  _edit_last: '2'
---
<a href="http://code.google.com/p/yara-project/">YARA</a> is a project I've been wanting to do something with for a while.  According to it's site, "YARA is a tool aimed at helping malware researchers to identify and classify malware samples".  Basically, write some antivirus signatures (or essentially regular expressions) and it can search a binary file for them.

HBGary's <a href="https://www.hbgary.com/community/free-tools/#fingerprint">FingerPrint</a> tool is also something I've also been wanting to mess with.  I described this tool in one of my previous posts titled <a href="http://0xdabbad00.com/2010/09/11/review-of-hbgarys-free-tools/">Review of HBGary’s free tools</a>.  Basically, it is just searching for strings in a binary file and using this info you can try to give you output that you might be able to use to cluster executables.

So HBGary's FingerPrint is a set of strings plus and engine for searching for those strings, and YARA is just an engine for searching for strings.  The difference between those engines is that YARA is fast and FingerPrint is slow in my testings.  So <b>I converted HBGary's FingerPrint rules to ones that can be used by YARA!</b>  Instead of taking roughly 7 seconds per binary using FingerPrint, you can use YARA and get the same info in under a tenth of a second using my rules.

<b><a href="http://0xdabbad00.com/wp-content/uploads/2011/01/yara_fingerprint.zip">Download here</a></b>

To use, simply extract those files, copy in the yara binary (I am using yara v1.4), and copy in a file you want to scan, and run:
<code>yara.exe -g -m fingerprints.yar file_to_scan.exe</code>

You can use the same rules with yara's other functionality (for Linux, or python).  fingerprints.yar just includes all the other files.  Also, it's important to note a few things:
<ul>
<li>This is incomplete.   I got bored and felt this wasn't really that cool, so you'll see in fingerprints.yar a TODO to add in msapi.cs, pe.cs, and strings.cs.  You can get all that info by running dumpbin from the Visual Studio, or another PE parser, so I didn't bother.  Really, almost all of this info you can get elsewhere, with few exceptions (some byte pattern searches, that result in lots of false positives though).  There are some other TODO's where I skipped things I couldn't do with YARA.
<li>My rules just look for the existence of strings, so I identify the existence of "send" in the string "caught while sending " for example, instead of ensuring that "send" has non-ascii on either side of it, so this will have more false positives than FingerPrint.
<li>FingerPrint itself has lots of false positives.  Some of it's rules are very loose, and I copied them exactly.
<li>My output looks a lot different than FingerPrint.  FingerPrint has some XML and pretty command-line output, whereas mine is more geared towards being fed into a second stage analysis application.  I even included weights as metadata with all the rules (even though all the weights are 1), and I also tagged all the rules which might be useful.
<li>This is much faster than FingerPrint. :P
</ul>

I'll admit, this is not the coolest or most useful thing ever, but it's my first use of YARA.
