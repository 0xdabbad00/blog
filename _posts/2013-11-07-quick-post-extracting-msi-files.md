---
layout: post
title: ! 'Quick Post: Extracting MSI files'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _edit_lock: '1390753394'
---
This is just a quick post to show how to extract MSI files.  Nothing exciting, but I thought I should start posting some things that might not be common knowledge.

Microsoft <a href="http://blogs.technet.com/b/srd/archive/2013/11/05/cve-2013-3906-a-graphics-vulnerability-exploited-through-word-documents.aspx">announced</a> this week CVE-2013-3906, a graphics vulnerability in Office and Lync, which is being actively exploited.  In an incident like this, Microsoft will create a <a href="https://support.microsoft.com/kb/2896666">Fixit</a>, which is a short-term solution to disable whatever functionality the exploit is abusing, often by setting a registry value.  It is interesting that Microsoft has built-in many kill switches into their products, which allow functionality to be disabled by setting certain registry values.

Microsoft says what this Fixit does.  It simply sets:
<code>HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Gdiplus\DisableTIFFCodec = 1</code>

This post will just show how to confirm that.

Download <a href="http://go.microsoft.com/?linkid=9840894">MicrosoftFixit51004.msi</a> (hash: 121797190c6c0bb386166b67565f0ce5) from <a href="https://support.microsoft.com/kb/2896666">https://support.microsoft.com/kb/2896666</a>.

You can use <a href="http://www.7-zip.org/">7-zip</a> to "decompress" the file.  7-zip will "decompress" .exe's into it's different sections (.data, .text., resources). 7-zip is amazing.  It's the robitussin of the file world.  Apply it to everything and liberally.

<a href="http://0xdabbad00.com/wp-content/uploads/2013/11/fixit_extraction.png"><img src="http://0xdabbad00.com/wp-content/uploads/2013/11/fixit_extraction-300x245.png" alt="" title="Fixit extraction with 7-zip" width="300" height="245" class="aligncenter size-medium wp-image-1254" /></a>

I use 7-zip on .MSI's so I can extract out the executables inside without needing to actually install the software.  In this case though the Fixit just has a bunch of directories for the <a href="http://msdn.microsoft.com/en-us/goglobal/bb964664.aspx">locale ids</a> (meaning this is the text that should be displayed when the MSI is run on a system configured for that language) and then some binary data.  The only file of interest there is <code>Binary.LogDll</code> which is a DLL file that seems to do some telemetry back to Microsoft, but that's not what this post is about.

As we saw, 7-zip gave us a lot of binary data.  It's helpful at this point to know how MSI's are created.  The 3 ways you'll probably end up creating an installer on Windows are to use:
<ul>
<li><a href="http://nsis.sourceforge.net/Main_Page">NSIS</a> from Nullsoft (yep, the makers of Winamp... "<a href="http://en.wikipedia.org/wiki/Nullsoft">Winamp, it really whips the llama's ass!</a>").  It's open-source but only makes .exe files.  You write a little script that says what files you want in your installer, how you want the installation process to work, and it takes care of the rest.
<li><a href="http://www.installshield.com/">InstallShield</a> which is garbage and saves everything in a binary format so you can't version control it, but since it's been around since the dawn of time, you'll probably run into it.
<li><a href="http://wixtoolset.org/">Wix</a> which is open-source and is used to make the installer for Visual Studio and many other products.  It was one of the first projects Microsoft released as open-source, back in 2004.  This is the tool you want to use to make installers.
</ul>

One cool thing about Wix is it has a "decompiler" of sorts to take a .msi and turn it into the script that was used to create it.  Standard decompiling issues apply: It won't work perfectly and what you get will still barely be readable.  Run:
<code>dark MicrosoftFixit51004.msi</code>

Most of the wix tools are named things like "candle", "heat", "smoke", "melt", etc. so the opposite of that is "dark".

You'll end up with a file called <code>MicrosoftFixit51004.wxs</code> with 859 lines to it.  Skimming through it, you'll see the line:
<code>
&lt;RegistryValue Id="ADDKEY1" Root="HKLM" Key="SOFTWARE\Microsoft\Gdiplus" Name="DisableTIFFCodec" Value="1" Type="integer" KeyPath="yes" /&gt;
</code>

So this does what we expected.  Maybe there is other stuff in the .msi, like that telemetry stuff, but I just wanted to show how you can do a little light reversing on .msi files.
