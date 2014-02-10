---
layout: post
title: Review of HBGary's free tools
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1294279748'
  _edit_last: '2'
---
Following up on my <a href="http://0xdabbad00.com/2010/09/11/review-of-mandiants-free-tools/">review</a> about <a href="http://www.mandiant.com/">Mandiant</a>'s tools (which I found were all not useful), I figured I'd check out <a href="https://www.hbgary.com/">HBGary</a>'s free tools, since I view the companies as being fairly similar, with HBGary seeming to be the more established player due to it's partnerships with companies such as McAfee (it's other partnerships with Palantir and Skout Forensics aren't very interesting, as those companies aren't very interesting).

HBGary has 4 <a href="https://www.hbgary.com/community/free-tools/">free tools</a>: FingerPrint, fget, FastDump, and FlyPaper.

<b>FingerPrint</b>
<a href="https://www.hbgary.com/community/free-tools/#fingerprint">Fingerprint</a> is a tool to for identifying features in executables that could assist in clustering malware into families, attributing various malware samples to the same group, or help to identify features that make an executable different within a group.  At least those are the uses I see for it, they don't really go into how it's useful anywhere in the writings I see from the company.  As discussed in one of hbgary's <a href="https://www.hbgary.com/press/hbgary%E2%80%99s-fingerprint-a-path-to-attribution/">press releases</a> "Malware creators have specific styles, use specific tools, and develop in specific environments in specific ways."  The tool is open source, so you can see the identifiers they are using.  Basically the tool is just looking for strings in an executable, which mostly correspond with the IAT imported functions, and then it associates some explanatory texts when it finds the strings.  For example, if it sees the text "IsDebuggerPresent" it outputs "Debugger Check", if it sees "wsock32.dll" or "ws2_32.dll" it outputs "Windows socket library".  It uses similar tests to determine the compiler used.

This tool is basically useful to create input to other tools, but no means of doing this is discussed.  So for example, you would want to take the output from FingerPrint and then use it to compare various binaries in order to perform clustering or associate similarity scores, but there is no discussion on how to do that (maybe I'll take a swing at this in another article).

<u>Conclusion</u>: By itself, this tool is not too useful.  It needs an analysis frontend.

<b>fget</b>
<a href="https://www.hbgary.com/community/free-tools/#fget">fget</a> is described as a tool that 
<blockquote>forensically extracts files from raw NTFS volumes on remote windows systems in your domain. This tool works over the network and can extract any file (including those that are locked and in-use) in a forensically sound manner, without altering target filetimes or attributes.</blockquote>
At first you might think this sounds like just some lame file copy tool, but it's actually bad-ass.  So getting a file without altering the filetimes or attributes is cool and useful, but not too exciting.  Getting a file that is currently locked and in-use though, is a very cool feature!  I decided to test that and was able to grab a copy of C:\pagefile.sys on my local Windows XP x64 system! Very cool and very fast (took a minute maybe to grab a 1.5GB file, which I guess is to be expected but I still sort of figured that with whatever trickery they must be doing it would take longer).  I also tried copying a file for which I removed all the ACL's on it, so I can't copy it or read it anymore, but fget still worked!  fget apparently works by parsing the actual NTFS volume, as opposed to going through any API calls.  That's quite impressive!

I did not try testing the tool to grab any remote files.

<u>Conclusion</u>: Very useful!  I wish I could replace my normal Windows copy and paste with this tool!

<b>FastDump</b>
<a href="https://www.hbgary.com/community/free-tools/#fastdump">FastDump</a> is a single exe that can grab the memory on a currently executing system.  I tried FastDump on a Win XP 32-bit image and used <a href="http://moonsols.com/product">MoonSols Windows Memory Toolkit</a> bin2dmp to convert that file to a dmp file so I could open it up in windbg.   I then ran FastDump on Windows XP x64 and it produced a big file quickly, but I have no way of knowing if it's correct, or anyway of analyzing the memory, because for that you need HBGary's Responder tool, which is not free, and the free version of Moonsols only supports 32-bit XP and Vista.

<u>Conclusion</u>: Easier to just use Moonsols.

<b>FlyPaper</b>
<a href="https://www.hbgary.com/community/free-tools/#flypaper">FlyPaper</a> is a tool to grab copies of anything that malware writes out.  The idea here is if some malware extracts out files and then deletes them, that flypaper will have grabbed copies.

<u>Conclusion</u>: Not super useful, and not too difficult to complex to create, but I guess it must have filled some need.

<b>Final Conclusion</b>: HBGary, <a href="http://0xdabbad00.com/2010/12/04/hbgary-is-small/">considering they are only 38 people</a>, has made some cool tools.  Their fget tool is definitely the most useful, but their FingerPrint tool, with some work, could also be great.   They definitely have some talented coders, and there free tools are good marketing material for their expertise.
