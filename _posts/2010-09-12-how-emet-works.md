---
layout: post
title: How EMET works
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1298732703'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
I'm interested in how easily <a href="http://blogs.technet.com/b/srd/archive/2010/09/02/enhanced-mitigation-experience-toolkit-emet-v2-0-0.aspx">EMET</a> is able to turn DEP, ASLR and other security features on in executables, so I decided to take a look.  I chose an executable, turned on <a href="http://technet.microsoft.com/en-us/sysinternals/bb896645.aspx">ProcMon</a> from Sysinternals, and used EMET to turn on the security on that executable.  I checked the executable afterward to ensure it's actually the same as before (so they're not bit fiddling).  I then looked through ProcMon's output, and found that it adds a registry key to <code>HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Custom\</code> with the name of the file, and with REG_QWORD values that are always the same.  For example, I have:
<code>HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Custom\firefox.exe\{e1c810aa-f7cc-4aaf-ada1-181863075f9b}.sdb = "8c 05 84 fb f0 4e cb 01"
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Custom\firefox.exe\{f8c4cc07-6dc4-418f-b72b-304fcdb64052}.sdb = "8c 05 84 fb f0 4e cb 01"</code>

Those GUID's then correlate back to keys at <code>HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\InstalledSDB\</code> which identify files at the following locations:
<code>C:\WINDOWS\AppPatch\Custom\Custom64\{e1c810aa-f7cc-4aaf-ada1-181863075f9b}.sdb
C:\WINDOWS\AppPatch\Custom\{f8c4cc07-6dc4-418f-b72b-304fcdb64052}.sdb</code>

I then googled for info on .sdb files, and came to the page for a tool called <a href="http://blogs.msdn.com/b/heaths/archive/2007/11/02/sdb2xml.aspx">sdb2xml</a>.  When I ran that on the .sdb file, I got an XML file showing each of the processes I had turned EMET on for, but I'll just show the portion relevant to firefox, along with the rest of the info:
[sourcecode language="xml"]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;IBM437&quot; standalone=&quot;yes&quot;?&gt;
&lt;SDB xmlns:xs=&quot;http://www.w3.org/2001/XMLSchema&quot; path=&quot;file.sdb&quot;&gt;
  &lt;INDEXES&gt;
    &lt;INDEX&gt;
      &lt;INDEX_TAG type=&quot;xs:short&quot;&gt;28676&lt;/INDEX_TAG&gt;
      &lt;INDEX_KEY type=&quot;xs:short&quot;&gt;24577&lt;/INDEX_KEY&gt;
      &lt;INDEX_BITS type=&quot;xs:base64Binary&quot; /&gt;
    &lt;/INDEX&gt;
    &lt;INDEX&gt;
      &lt;INDEX_TAG type=&quot;xs:short&quot;&gt;28691&lt;/INDEX_TAG&gt;
      &lt;INDEX_KEY type=&quot;xs:short&quot;&gt;24577&lt;/INDEX_KEY&gt;
      &lt;INDEX_BITS type=&quot;xs:base64Binary&quot; /&gt;
    &lt;/INDEX&gt;
    &lt;INDEX&gt;
      &lt;INDEX_TAG type=&quot;xs:short&quot;&gt;28679&lt;/INDEX_TAG&gt;
      &lt;INDEX_KEY type=&quot;xs:short&quot;&gt;24577&lt;/INDEX_KEY&gt;
      &lt;INDEXFLAGS type=&quot;xs:int&quot;&gt;1&lt;/INDEXFLAGS&gt;
      &lt;INDEX_BITS type=&quot;xs:base64Binary&quot; /&gt;
    &lt;/INDEX&gt;
  &lt;/INDEXES&gt;
  &lt;DATABASE&gt;
    &lt;OS_PLATFORM type=&quot;xs:int&quot;&gt;1&lt;/OS_PLATFORM&gt;
    &lt;NAME type=&quot;xs:string&quot;&gt;EMET Database&lt;/NAME&gt;
    &lt;DATABASE_ID_x0028_GUID_x0029_ type=&quot;xs:string&quot; baseType=&quot;xs:base64Binary&quot;&gt;{f8c4cc07-6dc4-418f-b72b-304fcdb64052}&lt;/DATABASE_ID_x0028_GUID_x0029_&gt;
    &lt;LIBRARY&gt;
      &lt;FLAG&gt;
        &lt;NAME type=&quot;xs:string&quot;&gt;EnableDEP&lt;/NAME&gt;
        &lt;FLAGS_PROCESSPARAM type=&quot;xs:long&quot;&gt;131072&lt;/FLAGS_PROCESSPARAM&gt;
      &lt;/FLAG&gt;
      &lt;SHIM&gt;
        &lt;NAME type=&quot;xs:string&quot;&gt;EMET Shim&lt;/NAME&gt;
        &lt;DLLFILE type=&quot;xs:string&quot;&gt;EMET.dll&lt;/DLLFILE&gt;
        &lt;INEXCLUDE&gt;
          &lt;INCLUDE /&gt;
          &lt;MODULE type=&quot;xs:string&quot;&gt;*&lt;/MODULE&gt;
        &lt;/INEXCLUDE&gt;
      &lt;/SHIM&gt;
    &lt;/LIBRARY&gt;
&lt;EXE&gt;
      &lt;NAME type=&quot;xs:string&quot;&gt;firefox.exe&lt;/NAME&gt;
      &lt;APP_NAME type=&quot;xs:string&quot;&gt;EMET Apps&lt;/APP_NAME&gt;
      &lt;EXE_ID_x0028_GUID_x0029_ type=&quot;xs:string&quot; baseType=&quot;xs:base64Binary&quot;&gt;{7e589b34-a304-44ef-b715-bb037cb5221f}&lt;/EXE_ID_x0028_GUID_x0029_&gt;
      &lt;MATCHING_FILE&gt;
        &lt;NAME type=&quot;xs:string&quot;&gt;*&lt;/NAME&gt;
      &lt;/MATCHING_FILE&gt;
      &lt;SHIM_REF&gt;
        &lt;NAME type=&quot;xs:string&quot;&gt;EMET Shim&lt;/NAME&gt;
        &lt;SHIM_TAGID type=&quot;xs:int&quot;&gt;236&lt;/SHIM_TAGID&gt;
      &lt;/SHIM_REF&gt;
      &lt;FLAG_REF&gt;
        &lt;NAME type=&quot;xs:string&quot;&gt;EnableDEP&lt;/NAME&gt;
        &lt;FLAG_TAGID type=&quot;xs:int&quot;&gt;214&lt;/FLAG_TAGID&gt;
      &lt;/FLAG_REF&gt;
    &lt;/EXE&gt;
	&lt;/DATABASE&gt;
  &lt;STRINGTABLE&gt;
    &lt;STRTAB_ITEM type=&quot;xs:string&quot;&gt;EMET Database&lt;/STRTAB_ITEM&gt;
    &lt;STRTAB_ITEM type=&quot;xs:string&quot;&gt;EnableDEP&lt;/STRTAB_ITEM&gt;
    &lt;STRTAB_ITEM type=&quot;xs:string&quot;&gt;EMET Shim&lt;/STRTAB_ITEM&gt;
    &lt;STRTAB_ITEM type=&quot;xs:string&quot;&gt;EMET.dll&lt;/STRTAB_ITEM&gt;
    &lt;STRTAB_ITEM type=&quot;xs:string&quot;&gt;*&lt;/STRTAB_ITEM&gt;
    &lt;STRTAB_ITEM type=&quot;xs:string&quot;&gt;EMET Apps&lt;/STRTAB_ITEM&gt;
    &lt;STRTAB_ITEM type=&quot;xs:string&quot;&gt;firefox.exe&lt;/STRTAB_ITEM&gt;
  &lt;/STRINGTABLE&gt;
&lt;/SDB&gt;
[/sourcecode]

This suggests that when an executable with the name "firefox.exe" gets run, the DLL "EMET.dll" gets loaded.  This file is at <code>C:\WINDOWS\AppPatch\EMET.dll</code>, and running it through IDA we see a call to an internal function labeled "aSetprocessdepp" which calls GetModuleHandle()/GetProcAddress() on "kernel32.dll"/"SetProcessDEPPolicy".  I didn't know such a function existed, but we can check it out in the MSDN at <a href="http://msdn.microsoft.com/en-us/library/bb736299%28VS.85%29.aspx">SetProcessDEPPolicy Function</a> and see that it just turns DEP on or off (duh!).  Just for fun, continuing to read the MSDN article, we can see that a process can also have it's DEP setting changed by use of the function <a href="http://msdn.microsoft.com/en-us/library/ms686880%28v=VS.85%29.aspx">UpdateProcThreadAttribute</a> or when creating the process in the call to <a href="http://msdn.microsoft.com/en-us/library/ms682425%28v=VS.85%29.aspx">CreateProcess</a> you can specify that same attribute list in the <a href="http://msdn.microsoft.com/en-us/library/ms686329%28v=VS.85%29.aspx">STARTUPINFOEX</a> structure.

Although EMET keys off of the name of the file, you can also use "file size, checksum, version, and date" according to the MSDN article on the <a href="http://msdn.microsoft.com/en-us/library/bb432182%28v=VS.85%29.aspx">Application Compatibility Database</a>.

Also, I found it odd that the .pdb file listed inside the EMET.dll is "c:\\src\\<b>redteam</b>\\src\\tools\\defense\\emet\\src\\razzle\\shimdll\\objfre\\i386\\EMET.pdb" when the EMET tool is produced by the <b>BlueHat</b> team.  What color are you!?!

I couldn't figure out how EMET is enabling the other security features (like ASLR).  I guess it's trickier then just making an API call.
