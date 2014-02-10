---
layout: post
title: How EMET works
categories: []
tags: []
status: publish
type: post
published: true
---
I'm interested in how easily <a href="http://blogs.technet.com/b/srd/archive/2010/09/02/enhanced-mitigation-experience-toolkit-emet-v2-0-0.aspx">EMET</a> is able to turn DEP, ASLR and other security features on in executables, so I decided to take a look.  I chose an executable, turned on <a href="http://technet.microsoft.com/en-us/sysinternals/bb896645.aspx">ProcMon</a> from Sysinternals, and used EMET to turn on the security on that executable.  I checked the executable afterward to ensure it's actually the same as before (so they're not bit fiddling).  I then looked through ProcMon's output, and found that it adds a registry key to `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Custom\` with the name of the file, and with REG_QWORD values that are always the same.  For example, I have:
`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Custom\firefox.exe\{e1c810aa-f7cc-4aaf-ada1-181863075f9b}.sdb = "8c 05 84 fb f0 4e cb 01"
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Custom\firefox.exe\{f8c4cc07-6dc4-418f-b72b-304fcdb64052}.sdb = "8c 05 84 fb f0 4e cb 01"`

Those GUID's then correlate back to keys at `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\InstalledSDB\` which identify files at the following locations:
`C:\WINDOWS\AppPatch\Custom\Custom64\{e1c810aa-f7cc-4aaf-ada1-181863075f9b}.sdb
C:\WINDOWS\AppPatch\Custom\{f8c4cc07-6dc4-418f-b72b-304fcdb64052}.sdb`

I then googled for info on .sdb files, and came to the page for a tool called <a href="http://blogs.msdn.com/b/heaths/archive/2007/11/02/sdb2xml.aspx">sdb2xml</a>.  When I ran that on the .sdb file, I got an XML file showing each of the processes I had turned EMET on for, but I'll just show the portion relevant to firefox, along with the rest of the info:
{% highlight xml %}
<?xml version="1.0" encoding="IBM437" standalone="yes"?>
<SDB xmlns:xs="http://www.w3.org/2001/XMLSchema" path="file.sdb">
  <INDEXES>
    <INDEX>
      <INDEX_TAG type="xs:short">28676</INDEX_TAG>
      <INDEX_KEY type="xs:short">24577</INDEX_KEY>
      <INDEX_BITS type="xs:base64Binary" />
    </INDEX>
    <INDEX>
      <INDEX_TAG type="xs:short">28691</INDEX_TAG>
      <INDEX_KEY type="xs:short">24577</INDEX_KEY>
      <INDEX_BITS type="xs:base64Binary" />
    </INDEX>
    <INDEX>
      <INDEX_TAG type="xs:short">28679</INDEX_TAG>
      <INDEX_KEY type="xs:short">24577</INDEX_KEY>
      <INDEXFLAGS type="xs:int">1</INDEXFLAGS>
      <INDEX_BITS type="xs:base64Binary" />
    </INDEX>
  </INDEXES>
  <DATABASE>
    <OS_PLATFORM type="xs:int">1</OS_PLATFORM>
    <NAME type="xs:string">EMET Database</NAME>
    <DATABASE_ID_x0028_GUID_x0029_ type="xs:string" baseType="xs:base64Binary">{f8c4cc07-6dc4-418f-b72b-304fcdb64052}</DATABASE_ID_x0028_GUID_x0029_>
    <LIBRARY>
      <FLAG>
        <NAME type="xs:string">EnableDEP</NAME>
        <FLAGS_PROCESSPARAM type="xs:long">131072</FLAGS_PROCESSPARAM>
      </FLAG>
      <SHIM>
        <NAME type="xs:string">EMET Shim</NAME>
        <DLLFILE type="xs:string">EMET.dll</DLLFILE>
        <INEXCLUDE>
          <INCLUDE />
          <MODULE type="xs:string">*</MODULE>
        </INEXCLUDE>
      </SHIM>
    </LIBRARY>
<EXE>
      <NAME type="xs:string">firefox.exe</NAME>
      <APP_NAME type="xs:string">EMET Apps</APP_NAME>
      <EXE_ID_x0028_GUID_x0029_ type="xs:string" baseType="xs:base64Binary">{7e589b34-a304-44ef-b715-bb037cb5221f}</EXE_ID_x0028_GUID_x0029_>
      <MATCHING_FILE>
        <NAME type="xs:string">*</NAME>
      </MATCHING_FILE>
      <SHIM_REF>
        <NAME type="xs:string">EMET Shim</NAME>
        <SHIM_TAGID type="xs:int">236</SHIM_TAGID>
      </SHIM_REF>
      <FLAG_REF>
        <NAME type="xs:string">EnableDEP</NAME>
        <FLAG_TAGID type="xs:int">214</FLAG_TAGID>
      </FLAG_REF>
    </EXE>
	</DATABASE>
  <STRINGTABLE>
    <STRTAB_ITEM type="xs:string">EMET Database</STRTAB_ITEM>
    <STRTAB_ITEM type="xs:string">EnableDEP</STRTAB_ITEM>
    <STRTAB_ITEM type="xs:string">EMET Shim</STRTAB_ITEM>
    <STRTAB_ITEM type="xs:string">EMET.dll</STRTAB_ITEM>
    <STRTAB_ITEM type="xs:string">*</STRTAB_ITEM>
    <STRTAB_ITEM type="xs:string">EMET Apps</STRTAB_ITEM>
    <STRTAB_ITEM type="xs:string">firefox.exe</STRTAB_ITEM>
  </STRINGTABLE>
</SDB>
{% endhighlight %}

This suggests that when an executable with the name "firefox.exe" gets run, the DLL "EMET.dll" gets loaded.  This file is at `C:\WINDOWS\AppPatch\EMET.dll`, and running it through IDA we see a call to an internal function labeled `aSetprocessdepp` which calls `GetModuleHandle()`/`GetProcAddress()` on `kernel32.dll`/`SetProcessDEPPolicy`.  I didn't know such a function existed, but we can check it out in the MSDN at <a href="http://msdn.microsoft.com/en-us/library/bb736299%28VS.85%29.aspx">SetProcessDEPPolicy Function</a> and see that it just turns DEP on or off (duh!).  Just for fun, continuing to read the MSDN article, we can see that a process can also have it's DEP setting changed by use of the function <a href="http://msdn.microsoft.com/en-us/library/ms686880%28v=VS.85%29.aspx">`UpdateProcThreadAttribute`</a> or when creating the process in the call to <a href="http://msdn.microsoft.com/en-us/library/ms682425%28v=VS.85%29.aspx">`CreateProcess`</a> you can specify that same attribute list in the <a href="http://msdn.microsoft.com/en-us/library/ms686329%28v=VS.85%29.aspx">`STARTUPINFOEX`</a> structure.

Although EMET keys off of the name of the file, you can also use "file size, checksum, version, and date" according to the MSDN article on the <a href="http://msdn.microsoft.com/en-us/library/bb432182%28v=VS.85%29.aspx">Application Compatibility Database</a>.

Also, I found it odd that the .pdb file listed inside the EMET.dll is `c:\src\redteam\src\tools\defense\emet\src\razzle\shimdll\objfre\i386\EMET.pdb` when the EMET tool is produced by the <b>BlueHat</b> team.  What color are you!?!

I couldn't figure out how EMET is enabling the other security features (like ASLR).  I guess it's trickier then just making an API call.
