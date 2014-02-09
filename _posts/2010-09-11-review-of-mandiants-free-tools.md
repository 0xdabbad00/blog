---
layout: post
title: Review of Mandiant's free tools
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1284324252'
  _edit_last: '2'
---
<a href="http://www.mandiant.com/">Mandiant</a> is one of companies I like to watch.  I follow their blog, called <a href="http://blog.mandiant.com/">M-unition</a>.  Jamie Butler works there now after he left <a href="http://www.hbgary.com/">hbgary</a>, where him and Greg Hoglund wrote <a href="http://www.amazon.com/Rootkits-Subverting-Windows-Greg-Hoglund/dp/0321294319/">Rootkits: Subverting the Windows Kernel</a> in 2005.  My current favorite rootkit book is <a href="http://www.amazon.com/Rootkit-Arsenal-Escape-Evasion-Corners/dp/1598220616/">The Rootkit Arsenal: Escape and Evasion in the Dark Corners of the System</a> by Bill Blunden, published in 2009, but Butler and Hoglund's rootkit book was and is an excellent resource for their time.

Mandiant offers 7 <a href="http://www.mandiant.com/products/free_software">free tools on their site</a> for various malware analysis and incident response tasks.  However, I'm only interested in 3 of them: IOCe, Memoryze, and Red Curtain.

<b>IOCe</b>
<a href="http://www.mandiant.com/products/free_software/ioce/">IOCe</a> (Indicators of Compromise Editor) is supposed to be a tool for describing some aspects of malware (such as it's filename and imports) as opposed to just writing byte sequences to scan for, and then somehow I'm guessing you can use these IOC's in conjunction with some other tool to actually scan a computer for the malware.  However, I'm honestly not too sure what this tool is, or how to use it, because there seems to be no documentation anywhere.  When you "install" it, it just adds some random files to "C:\Program Files\Mandiant\OpenIOC".  By random, I mean it adds .pdb files (why do I need debugger files?), a bunch of .bat files with the word "demo" in their name (that just spit out errors... cool "demonstration"), a "src" directory with no source code, and tons of XML and XSLT files, amongst other things.  I can't figure out how to use this tool, and thus I'm not really even sure what it does.  You also get a link from your Start menu, but the GUI that pop's up is badly aliased (the text is blurry), and I still can't figure out what the tool should be doing!

<u>Conclusion</u>: Not useful because I can't figure out how to use the tool or find any documentation, and I'm not even really sure what it is supposed to do.

<b>Memoryze</b>
<a href="http://www.mandiant.com/products/free_software/memoryze/">Memoryze</a> is described as being able to <i>"acquire and/or analyze memory images, and on live systems can include the paging file in its analysis."</i>.  Using this memory image, it can then supposedly identify the processes that are running, drivers loaded, rootkit hooks, and tons of other info.  Unfortunately, it only works on the following OS's:
<ul>
<li>Windows 2000 Service Pack 4
<li>Windows XP Service Pack 2 and and Service Pack 3 (32-bit)
<li>Windows 2003 Service Pack 2 (32-bit)
<li>Windows Vista Service Pack 1 and Service Pack 2 (32-bit)
<li>Windows 2003 Service Pack 2 (64-bit)
</ul>
Most of my systems 64-bit or Windows 7 now, so I had to find an old Windows XP SP2 to test this on.  It's a little kludgier than HBGary's <a href="https://www.hbgary.com/community/free-tools/#fastdump">FastDump</a> and <a href="http://moonsols.com/blog/9-moonsols-windows-memory-toolkit">Moonsols</a> in that you have to install Memoryze to the Program Files dir and it supplies a bunch of files, whereas those other options are just single binaries.  There are some batch scripts to make things easier, so I ran MemoryDD.bat which gave me a memory dump, and then used Moonsols bin2dmp.exe program to convert it a Windows dump file that I could then load into windbg and play around with it by checking out things like the processes running and everything else you can do with a dump file in windbg.

The other .bat files just extract some data from the live memory (or memory dump), such as a process listing with various info about each process which can all probably be found using windbg.

<u>Conclusion</u>: As a program to dump the RAM of a system, it's not very useful to me because it only works on a limited number of environments.  It's additional capabilities to extract some information from the memory images isn't too useful, as windbg can do all of this.  If I wanted to write a tool to extract info from various memory images and do something with it, this tool might be useful.  So although this tool obviously is a VERY difficult tool to write, there are better options out there such as HBGary's FastDump and Moonsols.

<b>Red Curtain</b>
<a href="http://www.mandiant.com/products/free_software/red_curtain/">Red Curtain</a> is described as <i>"free software for Incident Responders that assists with the analysis of malware. MRC examines executable files (e.g., .exe, .dll, and so on) to determine how suspicious they are based on a set of criteria [...] such as the entropy (in other words, randomness), indications of packing, compiler and packing signatures, the presence of digital signatures, and other characteristics to generate a threat "score." This score can be used to identify whether a set of files is worthy of further investigation."</i>

I'm a big fan of the concept of tools like this (such as HBGary's Fingerprint), but they aren't very cool in practice.  The idea is that if you're looking for 0-day malware on hosts, then you need an easy way to filter through large amounts of executables to find the executables that are most likely to be malware that you can then analyze further.  A/V software isn't going to be too useful, because the idea is you're looking for 0-day malware, and A/V software only tells you something is malware if it is sure it is.  It just gives you a binary response.  An analyst needs to get back a score, so they can look at only the top 1% or so of files that ranked as being likely to be malware, but the tool isn't sure.

In the case of Red Curtain, you just get back a set of mostly worthless columns for each executable describing the size, MAC times, if it's signed, entropy scores, entry point signature, a score, an anomaly count, and a link to more details.  Under the details you get some simple parsed info about the PE sections and the what functions are imported.  I tried the tool on a couple of executables, and also ran it on a simple UPX packed executable that I packed with the default settings.  In some cases, the Entry Point Signature identified the compiler used, but it didn't know what the UPX was, but it did give it a high(ish) score of 3.7, compared with most of the files I tested it on  (installers for various programs and other Mandiant tools) had a score of .111 to 1.025.  I ran it on my whole cygwin directory, and it did give some files scores as high as 3.131, but with no indication as to why it scored that high.  On my C:\Windows directory, the highest scored file was 1.525.  It didn't identify anything really useful or interesting from any of the files and the tool is sort of kludgy because I can't just tell it the directory to scan by typing in (or copying in) the path, I have to browse to the path I want to scan (the tool is completely GUI based).

Looking through the help guide (a .chm), the tool really doesn't have the ability to find much cool stuff.  It gives a listing of anomalies it looks for, and does describe why those anomalies are interesting, but the tool doesn't link to that info, so unless you know where to look in the help, you'd be pretty lost wondering why the details reports a file as having "checksum_is_zero".  It's a decent start to a tool, and seems to focus on some research into looking at entropy, but all in all it isn't something I'd ever use.

<u>Conclusion</u>: The tool is not useful.  No useful results are given.

<b>Final Conclusion</b>
Mandiant's free tools are free because they aren't useful.  These are decent starts to tools, but I'm guessing they just posted these as "free" so they can grab them more easily for demos.
