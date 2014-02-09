---
layout: post
title: A failed attempt at identifying a developer using data in the PE file format
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1388617805'
  _edit_last: '2'
---
<h3>Summary</h3>
This was a failed idea, but I feel it's important document your failures to save others time.  I tried to see if .exe files left any evidence of who the creator was.  Specifically, I wanted to know if the GUID in the PDB section identified the machine the binary was compiled on.  It does not, as best as I am able to determine.  This post hopefully saves others the trouble and lets them fact check me.

<h3>Introduction</h3>
Back in October, someone posted how they were able to <a href="https://madiba.encs.concordia.ca/~x_decarn/truecrypt-binaries-analysis/">compile a matching binary for TrueCrypt for Windows</a>.  What interested me most was the note in there that each time a binary is compiled, it is given a unique GUID that changes for every compilation, in what is known as the <a href="http://www.godevtool.com/Other/pdb.htm">RSDS section</a>, which is basically a reference to PDB debugging info.

I knew every PE file (.exe file) for Windows ends up being different due to compilation/link time stamps, but I hadn't thought about what else might be in there, and this looked promising.  Specifically, I was reminded of how the author of the <a href="http://en.wikipedia.org/wiki/Melissa_(computer_virus)">Melissa virus</a> was discovered due to a GUID in the .doc file for the virus <a href="http://www.zdnet.com/news/tracking-melissas-alter-egos/101974">matching the GUID used in other files he was known to have created</a>.  In that case, the GUID was the same, but GUIDs are dangerous for those practicing OPSEC.  Specifically, <a href="http://en.wikipedia.org/wiki/Universally_unique_identifier">Type 1 GUID</a>'s are "randomly" generated numbers based on the MAC address of the system and timestamp, so if the GUID in the PE file happened to be a Type 1, then you'd be able to know the MAC address for the developer's system.  Perhaps something similar happened for these GUID's in PE files. 

See <a href="http://www.ietf.org/rfc/rfc4122.txt">RFC 4122</a> for more info on GUID formats.

Good OPSEC tradecraft for developers entails compiling binaries without the debug references.  For example, the sloppy developer that linked together the pieces of Stuxnet forgot to do this, resulting in the full path to the project folder ("b:\\myrtus\\src\\objfre_w2k_x86\\i386\\guava.pdb") being written to the binary and this naming scheme correlated <a href="http://media.blackhat.com/bh-dc-11/Parker/BlackHat_DC_2011_Parker_Finger%20Pointing-Slides.pdf">a few pieces of that puzzle together</a>.  Sometimes the developer uses an identifying username for their dev system and this ends up in that path string.

Only a small subset of the developers of the world need to practice good OPSEC.  It's only those developers that wish to remain anonymous, such as malware developers, authors of tools to help citizens in naughty countries, and a few others.

So anyway, my hypothesis was that this GUID may be used to identify developers in some way.  Given that a security company can't do much with a MAC address to identify a system in the world, I was more interested in if a nation state with greater resources could do such a task.  

Every once in a while you have to question your assumptions in life and think "What if what I thought was true was false, and how can I confirm that?" (that the UUID might identify a system) and "What if an adversary had far greater resources than I can imagine?" (the ability to find a system given a MAC address).  Also, every once in a while you should just reverse some random thing to see if you can figure out anything interesting about it, and if nothing else, it keeps your skills sharp.

<h3>What I'm looking for</h3>
In Visual Studio 2005 and up, all builds of binaries include this reference to the PDB file and a random GUID within the RSDS section.  Even Release builds include this.  You can change a project setting though to disable this.

<h3>Reversing</h3>
I used Visual Studio 2013, and created a simple project.  I turned on procmon, and built the project and recovered the build command.  I then brought up a command-line, ran vcvars.bat to create a Visual Studio environment for this command-line terminal, and ran that same build command and ensured I could build binaries without using Visual Studio, as this just simplifies things for me.  In this case, I had a project called <tt>recursive</tt> at <tt>C:\Users\user\Documents\Visual Studio 2013\Projects\recursive</tt>, and after running <tt>"C:\Program Files\Microsoft Visual Studio 12.0\VC\vcvarsall.bat"</tt>, I was able to build the project using the command: <pre>"C:\Program Files\Microsoft Visual Studio 12.0\VC\bin\link.exe" /ERRORREPORT:QUEUE /OUT:"C:\Users\user\Documents\Visual Studio 2013\Projects\recursive\Release\recursive.exe" /INCREMENTAL:NO /NOLOGO kernel32.lib user32.lib gdi32.lib winspool.lib comdlg32.lib advapi32.lib shell32.lib ole32.lib oleaut32.lib uuid.lib odbc32.lib odbccp32.lib /MANIFEST /MANIFESTUAC:"level='asInvoker' uiAccess='false'" /manifest:embed /DEBUG /PDB:"C:\Users\user\Documents\Visual Studio 2013\Projects\recursive\Release\recursive.pdb" /SUBSYSTEM:CONSOLE /OPT:REF /OPT:ICF /LTCG /TLBID:1 /DYNAMICBASE /NXCOMPAT /IMPLIB:"C:\Users\user\Documents\Visual Studio 2013\Projects\recursive\Release\recursive.lib" /MACHINE:X86 /SAFESEH recursive\Release\recursive.obj recursive\Release\stdafx.obj</pre>

Using windbg, I attached to <tt>cmd.exe</tt> and set a breakpoint on <tt>CreateProcessW</tt> (I also set one on <tt>CreateProcessA</tt> but only the wide-char version ever gets used).  Then I ran my <tt>link</tt> command.  I stepped out of the <tt>CreateProcessW</tt> call and attached to the <tt>link</tt> process in a separate windbg process.

By looking at the different functions (you can do this with IDA, or in Windbg using <tt>x link!</tt>) and setting breakpoints, and looking in IDA where the string "RSDS" is being used, I managed to figure out the following:
<ul>
<li>The string "RSDS" is written by <tt>IMAGE::WriteDebugInfo</tt> but the GUID has already been generated at some earlier point.
<li><tt>IMAGE::BuildImage</tt> calls <tt>IMAGE::Pass2</tt> which calls <tt>IMAGE::WriteDebugInfo</tt>
<li>The GUID ends up at <tt>ecx+22c</tt>, where ecx maintains some sort of structure throughout the entire <tt>link</tt> process.
<li>By setting a break on any writes to the address for <tt>ecx+22c</tt>, I figured out the GUID gets written by <tt>link!DBG_QuerySignature2</tt>
<li>The GUID is retrieved by making an RPC call into <tt>mspdbsrv.exe</tt> in the function <tt>mspdbcore!PDB1::QuerySignature2</tt>, but the GUID already has been generated somewhere in <tt>mspdbsrv.exe</tt> by the time this is called.
<li>I set break-point on <tt>mspdbcore!PDB1::PDB1</tt>, expecting this to an important constructor for my needs, and then set a break-on-write on where I expected the GUID to end up, based on my super sloppy idea of looking at the size of the memory allocations for one of size 0x26000 which I noticed was where the GUID ended up, and set a break-on-write on the offset within that memory region that I expected the GUID to be written.
<li>That idea worked, and my break-point hit within the function <tt>RPCRT4!rc4</tt> which is applying the RC4 encryption algorithm from within <tt>RPCRT4!GenerateRandomNumber</tt>, which is in <tt>RPCRT4!UuidCreate</tt>, which is exported by that DLL (<tt>RPCRT4.dll</tt>), and I'm kicking myself for not reading the imports/exports of the loaded modules earlier, because that would have saved me TONS of time on this.
</ul>

<h3>Identifying a system with UuidCreate?</h3>
According to the MSDN, <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/aa379205(v=vs.85).aspx"><tt>UuidCreate</tt></a> "generates a UUID that cannot be traced to the ethernet address of the computer on which it was generated. It also cannot be associated with other UUIDs created on the same computer."

However, doing some more research, led to <a href="http://www.oehive.org/book/export/html/457">this page</a>, which claims that <tt>UuidCreate</tt> worked differently on Windows 95/98/NT4!  However, it's probably very unlikely/impossible that someone is using Visual Studio 2005 or a more recent version on Windows 98 or less.

I also saw ReactOS had coded this function <a href="http://code.reactos.org/browse/reactos/trunk/reactos/dll/win32/rpcrt4/rpcrt4_main.c?r=61437#to283"><tt>reactos/dll/win32/rpcrt4/rpcrt4_main.c</tt></a> to call <tt>RtlGenRandom</tt>, but this isn't what was happening in the code.  <tt>UuidCreate</tt> calls an internal function called <tt>GenerateRandomNumber</tt> which does have the ability to call <tt>RtlGenRandom</tt> (referred to in IDA and online in some places as <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/aa387694(v=vs.85).aspx"><tt>SystemFunction036</tt></a>), but this call does not get exercised.  Instead there is a call to <tt>rc4_safe_select</tt> and then <tt>rc4_safe</tt>.  

If you know Russian, or can stumble through Google's translation, you can read <a href="http://www.gotdotnet.ru/blogs/denish/1965/">http://www.gotdotnet.ru/blogs/denish/1965/</a>, which has an amazingly accurate reversing of the <tt>UuidCreate</tt> and cryptanalysis of it, but it seems to require they dump the memory of the process and have multiple copies of the UUID, so I've decided that given that we will only have a single UUID (and obviously no memory dump of the developer's <tt>mspdbsrv.exe</tt> process) that I'm not going to be able to figure out any way to identify the developer's system.

<h3>Conclusion</h3>
I don't believe the UUID in the RSDS/PDB section of a PE file can be used to identify the developer for that binary.  However, due to the file path to the .pdb file existing there (which identifies the project name and often the username), a developer practicing OPSEC should set their linker settings to avoid having the RSDS section in their binary.



