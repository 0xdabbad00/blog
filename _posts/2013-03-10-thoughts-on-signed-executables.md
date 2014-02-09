---
layout: post
title: Thoughts on signed executables
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1363032352'
  _edit_last: '2'
---
In thinking about making an application to do white-listing on Windows, one of the first questions you have is how do you identify what to trust?  I played around with Faronic's <a href="http://www.faronics.com/en-uk/products/anti-executable/enterprise/?">Anti-Executable</a> software a little, and although they do a lot of what I want, and their personal tool is well-priced at $35, the trust issue is a problem.  When you install the tool, it just trusts everything on your system.  To some degree, you have to do this, because for the type of tool this is, the assumption is that you are running on clean system.  However, it doesn't allow you to do anything with the signers of signed binaries, so when I updated Google Chrome, it asked me if I wanted to trust the new binaries, which is the purpose of the tool, but it would be nice if it told me "You've trusted binaries from this signer before".  You do have the ability to look at the list of binaries it trusts, but it's mostly just the filenames and paths.  When it comes across a new file you just get some info parsed from the PE file.
<a href="http://0xdabbad00.com/wp-content/uploads/2013/03/anti_exe-dialog.png"><img src="http://0xdabbad00.com/wp-content/uploads/2013/03/anti_exe-dialog-300x236.png" alt="" title="Anti-Executable Dialog" width="300" height="236" class="aligncenter size-medium wp-image-897" /></a>

You can then try to find out more about the files from Faronic's site, but most files I tried were unknown, and for known files, it doesn't show much.
<a href="http://0xdabbad00.com/wp-content/uploads/2013/03/anti_exe-known_file.png"><img src="http://0xdabbad00.com/wp-content/uploads/2013/03/anti_exe-known_file-300x185.png" alt="" title="Anti-Executable known file information" width="300" height="185" class="aligncenter size-medium wp-image-898" /></a>

I have assumed that in this day and age, most binaries, from major corporations, are signed.  A white-listing product, should thus allow you take advantage of this and allow you to trust that signer, so that any new binaries on your system that come from that signer can be pre-trusted.  This is one of the features of AppLocker.

You should also have more information about files.  Instead of asking me if I want to trust a file and giving me info about the company name stored in the binary (easy to fake), and the file path and name, it should use the Internet to find out some info for me.

CrowdStrike recently released a tool called <a href="http://www.crowdstrike.com/blog/free-community-tool-crowdinspect/index.html">CrowdInspect</a> that checks currently running files against VirusTotal and the Team Cymru's Malware Hash Registry.  That's a tool for 2013.  Use the Internet hivemind to help me make decisions.

So what I want in the tool help me make decisions is to check the file against VirusTotal, help me identify where it might have come from, and for anything that is unknown I should be able to upload the file somewhere where it's trust can be further evaluated, or at least recorded so I can later identify points of infection/intrusion (more of an enterprise purpose).

But before I start coding, I need to check my other assumptions, can I make trust decisions based on the signers of binaries?  Stuxnet was well-known for having been signed by the legitimate certificates of JMicron and Realtek, so I don't necessarily want to 100% trust something based on it's signer, but can I use this concept at all?  How many binaries are signed?  How many signers are there?  I think for determining which signers I trust (and possibly all trust issues for binaries), I can use something like <a href="http://convergence.io/">Convergence</a> does for SSL certificates.  But first, let's see what is on my system.

I used Didier Stevens' <a href="http://blog.didierstevens.com/programs/authenticode-tools/">AnalyzePESig</a> tool to look at the binaries.  I used this tool because it's open-source so I could modify what it output to my liking.

I scanned my Windows 7 system (a VM with some development tools) for all files that begin with an "MZ" header, and then checked if they were signed and by who.  I found there were 36 signers, which seems fairly reasonable to keep track of.  There were 23992 executables (these end up being .exe, .dll, .sys, and some other files), of which only 1962 (8.2%) were not signed.  It is important to note that not all of these are unique because Windows caches and copies files in various places.  These numbers could be completely different from system to system as well, but you have to start with some data.  For the unsigned binaries, a lot of files came from "C:\Windows\assembly" (915 files), "C:\Windows\sxs" (117 files), "C:\Program Files\Git" (310 files, basically a minimal cygwin install), and other various locations.

For the signers, the breakdown looks like this (I got tired as I was doing some manual effort to correlate these things, and honestly, there is a lot more analysis that needs to be put into this for this to be very meaningful, but the main point is that a lot of stuff is signed by Microsoft already):
{% highlight text %}
  Count, Thumbprint,                             , Subject name
  11820, 02eceea9d5e0a9f3e39b6f4ec3f7131ed4e352c4, C=US, S=Washington, L=Redmond, O=Microsoft Corporation, OU=MOPR, CN=Microsoft Windows
   4925, 018b222e21fbb2952304d04d1d87f736ed46dea4, C=US, S=Washington, L=Redmond, O=Microsoft Corporation, OU=MOPR, CN=Microsoft Windows
   2520, 9617094a1cfb59ae7c1f7dfdb6739e4e7c40508f, C=US, S=Washington, L=Redmond, O=Microsoft Corporation, OU=MOPR, CN=Microsoft Corporation
    951, d57fac60f1a8d34877aeb350e83f46f6efc9e5f1, C=US, S=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation
    831, 9e95c625d81b2ba9c72fd70275c3699613af61e3, C=US, S=Washington, L=Redmond, O=Microsoft Corporation, OU=MOPR, CN=Microsoft Corporation
    296, 564e01066387f26c912010d06bd78d3cf1e845ab, C=US, S=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation
    141, 06c92bec3bbf32068cb9208563d004169448ee21, C=US, S=California, L=Mountain View, O=Google Inc, OU=Digital ID Class 3 - Java Object Signing, CN=Google Inc
    135, d468faeb5190bf9decd9827af470f799c41a769c, C=US, S=Washington, L=Redmond, O=Microsoft Corporation, OU=MOPR, CN=Microsoft Corporation
     73, 8aed552a1387870a53f5f8aee17a3761232a4609, C=US, S=California, L=Mountain View, O=Google Inc, OU=Digital ID Class 3 - Netscape Object Signing, CN=Google Inc
     66, 57e82e9da631a768d8890e0a0b85381e3cb06d2e, C=US, S=California, L=Palo Alto, O="VMware, Inc.", OU=Digital ID Class 3 - Microsoft Software Validation v2, OU=Marketing, CN="VMware, Inc."
     64, 10622c76f18897e95222a888556843f4ce7e6aca, C=DE, S=Berlin, L=Berlin, O=ThinPrint GmbH, OU=Digital ID Class 3 - Microsoft Software Validation v2, CN=ThinPrint GmbH
     57, a25800bb7577f5854b3823b82228d94140d0244e, C=US, S=Washington, L=Redmond, O=Microsoft Corporation, CN=Microsoft Corporation
     18, 0bc8249c29e2c5ee53abf5c233c0e7ff90f582f5, C=DE, O=Open Source Developer, CN=Sven Strickroth - Open Source Developer, E=mail@cs-ware.de
     16, 93859ebf98afdeb488ccfa263899640e81bc49f1,
     15, 665f9ee40c024625ffec8d0102a3946270a98dd8,
     13, 597f04600b965bbf8a011df89dd0403ed50c44e0,
     12, 3bf0735c54918aec95062d61dc387c36b2643bb8,
     10, bca0a72fbf7c3a6666cbb15eeb450a3ae6f14a48,
      9, 98483cc3cf08e666f188383a30aa00c49521617b,
      7, 54e57dc08f1298601a22d6ac5278d472d3bacb56,
      6, 6ebdfb3cf5d0a9691f8700f96bb16d62d344f229,
      5, b0ddf7c3098a9d2261e6fa25a36fa50dccf63c36,
      5, 8e1354ff462a548bafe42c3214472dd18e5cd931,
      5, 2d39f75fa6f6c06a3cb23c7bc2bf874cfe6ba0af,
      4, 617c3b8c0c6d7808ad7b116ded7a271ef2ded82b,
      4, 282d9806c3df7345929f64f5895ef2ea4ac29302,
      4, 0b96f3350e96268136c9118e36c91232e2511222,
      3, c464d7b441a7e78ec9e5e0a1d4602e1e95fda0f3,
      3, a8a4f672d2a011189d2989e137be4a89080dc757,
      3, 5c616dc011e309dfcd15c0ea32494186654a2cdc,
      3, 190352e3ab6dc702e6ab1e5296753efda57c4480,
      2, ca45aba93f7d68104ae730bf99d0f1f0aebdaf3b,
      1, 90c22db300f44ec79beab4662bb77ed1e81843bc,
      1, 8849d1c0f147a3c8327b4038783aec3e06c76f5b,
      1, 1e039f8c2bcb0d13fc5459957ecc5b9b7a271041,
      1, 1017a37c5edfd1da9d3a435ea7767d5e3a5a8daa,
{% endhighlight %}
