---
layout: post
title: OpenHIPS project decisions
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1300655837'
  _edit_last: '2'
---
<b>Hosting</b>
I originally hosted OpenHIPS at <a href="http://code.google.com/">Google Code</a>, which is pretty common, has a great interface for browsing other people's code projects, and everyone with a gmail account (meaning everyone) can use it.  Unfortunately, the uptime of the site was pretty bad.  I never would have thought anything by google would be down, but for hours at a time I would be unable to check-in code, so I had to switch to something more stable.  I looked at my other options, like github, but ultimately decided I wanted to try out something with some project management features, so I chose <a href="http://www.assembla.com/">Assembla</a> which is free for open-source project.  Unfortunately, I haven't yet used these extra features much yet.
<i>Winner: <a href="https://www.assembla.com/wiki/show/openhips/">Assembla</a></i>

<b>Installers</b>
One of the most important benefits of OpenHIPS will be it's ease-of-use.  As such, I need an easy to use installer.  There are really only two choices for making installers for free for Windows as far as I'm concerned, <a href="http://nsis.sourceforge.net/Main_Page">NSIS</a> or <a href="http://wix.sourceforge.net/">Wix</a>.  I've used both, and nowadays I use Wix for everything, including OpenHIPS.  One of the biggest reasons for using Wix is it is the only free option at all for creating .msi files, which are the only types of files you can deploy through Group Policy in managed enterprises.  This isn't needed for this project, but in other work I do, this is a requirement, so I'm familiar with this tool.  The other reason is that it is made by Microsoft and thus is integrated with Windows to do things like automatically create restore points, add your project to Add/Remove programs, and the uninstaller is created automatically, because you just need to run:
<code>msiexec /x installer.msi</code>
Although I like that restore points are automatically created for you with .msi files, the downside is that the install takes 10-15 seconds, which makes the install seem sketchy, because people think "What on earth is it doing that is taking so long?".

<b>Languages</b>
If you want to write security programs, you have only one choice: You must use C, possibly with some assembly.  However, it is my belief that you should try to use a higher level language as much as possible, especially if you plan on doing any GUI work, so OpenHIPS also has components written in C#.

<b>Components</b>
There are actually four executable files that will be run on the systems OpenHIPS is installed to.  There are two DLL's written in C, and two files written in C#.

<tt>ohipsp32.dll</tt> (OpenHIPS Protector 32-bit) is a recompilation, and slightly modified version, of <a href="http://blog.didierstevens.com/programs/heaplocker/">HeapLocker</a>.  This spawns some threads that make various checks, and allocs memory in various places that are expected to be hit if an exploit tries to gain execution.

<tt>ohipsfs32.dll</tt> (OpenHIPS First Stage 32-bit) is loaded into every process via the AppInit registry key.  Once loaded into a process, this will determine if it should then load in the Protector dll.  Using the AppInit key can cause issues if your DLL has a lot of imports, so this First Stage has a minimal set of imports, and just checks if it should load in the DLL that does the real work (and has lots of imports, so it's load is delayed).  This code is based largely on the project <a href="http://blog.didierstevens.com/2009/12/23/loaddllviaappinit/">LoadDLLViaAppInit</a> from Didier Stevens.

<tt>ohipsp64.dll</tt> and <tt>ohipsfs64.dll</tt> are the 64-bit versions of the above DLL's, but currently are unused.

<tt>ohipssvc.exe</tt> (OpenHIPS Service) is coded in C# and is a registered service.  You can run <code>sc query openhips</code> and see it running.  Currently, all this does is watch over the AppInit key to make sure nothing overwrites our removes our entry there.  As a service this runs with LocalSystem level privileges which will likely be useful later.

<tt>ohipsui.exe</tt> (OpenHIPS User-Interface) is coded in C# and is simply that lame little taskbar icon, which when double-clicked opens up the Settings form that allows you to see the settings.  This is started for each user via the Run key.

<b>Start-up</b>
In the above discussion you may have noticed that there are 3 ways each of the different components are obtaining execution after installation: AppInit registry key, Run key, and Windows services.  If you want to start-up with privileges, a Windows service is your only option.  If you want to start-up every time a user logs in, there are lots of options but using the Run key is one of the cleanest.  Another option, which I may do in the future is to use the service to determine when a user has logged in, and then start the UI in the context of that user.  Finally, if you want your code to load in specific processes when they start, you can either use the AppInit key, or you can somehow watch for processes to be started (various ways of doing this, with the best being a kernel driver that uses <a href="http://www.osronline.com/DDKx/kmarch/k108_5lwy.htm">PsSetCreateProcessNotifyRoutine</a> described in <a href="http://www.codeproject.com/KB/threads/procmon.aspx">http://www.codeproject.com/KB/threads/procmon.aspx</a>) and then inject code into that processes.  I actually started off down that path sort of, but it didn't seem worth the effort (no gain for more pain).

