---
layout: post
title: ! 'Finding slop: Common Windows apps still without DEP and ASLR'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1356121459'
  _edit_last: '2'
---
<b>tl;dr: Even common Windows apps with massive user bases are still horrible about using even the most basic security features.</b>  Please use <a href="http://www.microsoft.com/en-us/download/details.aspx?id=30424">EMET</a> on Windows to force these protections.

<b>UPDATE 2012.12.07</b>:<i> Got referenced on <a href="http://www.h-online.com/security/news/item/Many-popular-Windows-programs-have-insufficient-protection-1764311.html">The H</a>!  I did some follow up research on <a href="http://0xdabbad00.com/2012/12/07/dep-data-execution-prevention-explanation/">DEP</a>, that shows that DEP is actually not need on DLL's or 64-bit code, but I do feel, even for those files that don't need it, that if they don't have the NXCOMPAT bit set it probably indicates something awkward with their build system.</i>

<br><br><br>
DEP and ASLR are two simple protections every Windows binary should be implementing.  For a developer to use them, all they need to do is compile with any version of Visual Studio since 2005, and it should be the default setting.  If you are compiling Windows code, you should be using Visual Studio, and even if you use a different compiler, it should still have options for DEP and ASLR.  Enabling DEP and ASLR, I believe, is the <b>easiest</b> thing a developer can do to improve the security of their code.  Obviously, you can still have DEP and ASLR and still have the buggiest, sloppiest, most insecure code, but all you need to do is turn on two linker settings and instantly memory corruption exploits become much more difficult to exploit.

<b>A developer that does not enable DEP and ASLR is like a writer that does not use spell check.</b>  You can be a great writer and never misspell any words, but why not take advantage of something that is free, so easy to use, and would save you so much remorse later if you did make a mistake? Note: I am not a writer.  Also, I love many of these apps.  I use them.  I just wish they had DEP and ASLR enabled to give myself a little more piece of mind.

Using my <a href="http://icebuddha.com/slopfinder.htm">SlopFinder</a> project you can easily see which of your favorite apps have it enabled.  All of the executables, except for drivers (.sys files), should have it enabled.

Let's look around my Windows 7 x64 system and see what we find, and hand out grades to all the apps. <b>These grades are all based <u>only</u> on if DEP and ASLR are used, and how important the files are that are missing them.</b>  I only looked at the files in the directory associated with the app, so if the installer happens to install or use executables anywhere else that don't implement DEP and ASLR, then I will have missed those.  To put this another way, <b>this is not a very thorough analysis and my choice in grading is not based on any calculation</b>.
<ul>
<li><b>Java 7 (SE Runtime Environment build 1.7.0_09-b05): A+</b>
0/84 unprotected executables
Well done! All the binaries are protected.
<li><b>Google Chrome v23.0.1271.95 m: A</b>
3/143 executables unprotected
I don't think the unprotected executables are that important.  Apparently xinput1_3.dll is only loaded if you try to <a href="http://code.google.com/p/chromium/issues/detail?id=147642">control Chrome via an xbox controller</a> (what?).  I should fail it for including something so random in the binary and deciding that this is worth any security risk, but I'm trying to base this grade solely on the use of DEP and ASLR.  The two other unprotected executables seem to just be installers.
<ul>
<li>No DEP, No ASLR: /Google/Chrome/Application/23.0.1271.95/xinput1_3.dll
<li>No ASLR: /Google/Update/Download/{4DC8B4CA-1BDA-483E-B5FA-D3C12E15B62D}/23.0.1271.95/
23.0.1271.95_23.0.1271.91_chrome_updater.exe
<li>No ASLR: /Google/Update/Download/{8A69D345-D564-463C-AFF1-A69D9E530F96}/23.0.1271.95/
23.0.1271.95_chrome_installer.exe
</ul>
<li><b>Adobe Reader X v10.1.4: B</b>
4/78 executables unprotected
<ul>
<li>No DEP, No ASLR: /Reader 10.0/Reader/ccme_base.dll
<li>No DEP, No ASLR: /Reader 10.0/Reader/cryptocme2.dll
<li>No DEP, No ASLR: /Reader 10.0/Reader/icudt40.dll
<li>No DEP, No ASLR: /Reader 10.0/Reader/reader_sl.exe
</ul>
<li><b>Foxit Reader v4.3.0.01110: C</b>
3/4 executables unprotected
The main binary is protected, but nothing else.
<ul>
<li>No DEP, No ASLR: Foxit Software/Foxit Reader/Uninstall.exe
<li>No DEP, No ASLR: /Foxit Software/Foxit Reader/plugins/FoxitReaderOCX.ocx
<li>No DEP, No ASLR: /Foxit Software/Foxit Reader/plugins/npFoxitReaderPlugin.dll	
</ul>
<li><b>Firefox v11.0: C</b>
13/34 executables unprotected
Why so many unprotected from DEP?  Many of these unprotected DLLs are loaded by default.  I need to check if it's ok for the DLL's to not have DEP.
<ul>
<li>No DEP: /Mozilla Firefox/freebl3.dll
<li>No DEP: /Mozilla Firefox/nspr4.dll
<li>No DEP: /Mozilla Firefox/nss3.dll
<li>No DEP: /Mozilla Firefox/nssckbi.dll
<li>No DEP: /Mozilla Firefox/nssdbm3.dll
<li>No DEP: /Mozilla Firefox/nssutil3.dll
<li>No DEP: /Mozilla Firefox/plc4.dll
<li>No DEP: /Mozilla Firefox/plds4.dll
<li>No DEP: /Mozilla Firefox/smime3.dll
<li>No DEP: /Mozilla Firefox/softokn3.dll
<li>No DEP: /Mozilla Firefox/ssl3.dll
<li>No DEP, No ASLR: /Mozilla Firefox/plugins/npdeployJava1.dll
<li>No DEP, No ASLR: /Mozilla Firefox/uninstall/helper.exe
</ul>
<li><b>Internet Explorer 9 (32-bit) v9.0.8112.16421: D</b>
2/16 executables unprotected
The main binary is unprotected from DEP.  Likely for backwards compatibility with sloppy plugins.
<ul>
<li>No DEP: /Internet Explorer/ieinstal.exe
<li>No DEP: /Internet Explorer/iexplore.exe
</ul>
<li><b>Internet Explorer 9 (64-bit) v9.0.8112.16421: D</b>
3/24 executables unprotected
A whole different set of executables is unprotected from DEP in the 64-bit version.  That doesn't make sense.  I think all 64-bit apps are by default protected with DEP though.  Need to check.
<ul>
<li>No DEP: /Internet Explorer/msdbg2.dll
<li>No DEP: /Internet Explorer/pdm.dll
<li>No DEP: /Internet Explorer/sqmapi.dll
</ul>
<li><b>VMWare Workstation 7 (7.1.2 build-301548): F</b>
183/186 unprotected executables
Seriously!?? The only protected binaries are 3 Microsoft run-time files.  I'm not going to waste my time listing all the 183 unprotected binaries.
<li><b>7-zip [64] 4.65: F</b>
7/7 unprotected executables
<li><b>putty 0.62: F</b>
6/6 unprotected executables
All executables as part of the full putty.zip are unprotected.
<li><b>Cygwin: F</b>
Basically everything is unprotected.  Randomly some have ASLR enabled.
<li><b>TortoiseSVN: F</b>
All executables unprotected
<li><b>Spotify 0.8.5.1333: F</b>
3/5 executables unprotected
I think they just got lucky with 2 DLL's that are protected are from a 3rd party maybe.
<li><b>Dropbox: F</b>
7/7 executables unprotected
</ul>
