---
layout: post
title: Review of Microsoft SDL Tools
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1295745437'
  _edit_last: '2'
---
After seeing the article <a href="http://www.h-online.com/security/news/item/Tool-to-track-security-sensitive-changes-to-Windows-1172093.html">"Tool to track security-sensitive changes to Windows"</a>, I've decided to take a look at <a href="http://www.microsoft.com/security/sdl/default.aspx">Microsoft SDL</a> (Software Development Lifecycle) tools, from the perspective of how they can help me evaluate my own C code and other's.  I won't review all the tools, as some of the  tools are for .NET apps, and other "tools" are boring design and architecture "templates".  In most cases the templates appear to be some type of Visio template you can use to design things.  Boring, and also you can't use it with Visual Studio Express.

<b>banned.h</b>
This is a just a header file with a bunch of <tt>#pragma deprecated</tt> lines for functions like <tt>strcpy</tt>.  Visual Studio already gives you warnings for all these when you compile, so not interesting or useful.

<b>Code Analysis for C/C++</b>
Requires "Visual Studio Team System Development Edition", so not helpful for me, but based on the video it seems to be a compilation step that performs some extra checks on what values you are actually passing various Windows functions, and gives you warnings about buffer overflows or unsafe arguments.  So this would be nice if I could use it with VS Express.

<b>SDL Regex Fuzzer</b>
Allows you to give it a regular expression and it will tell you if this is susceptible to a DoS.  The only way this can happen is if you start doing groupings within the same expression... basically advanced expressions that I don't think anyone would ever use, or it would be obvious that this is going to cause issues without having to fuzz it and use a special tool, and worse case scenario, you get a DoS.  Boring.  Not useful.

<b>BinScope Binary Analyzer</b>
BinScope "is a verification tool that analyzes binaries to ensure that they have been built in compliance with the SDL requirements and recommendations. BinScope checks that SDL-required compiler/linker flags are being set, strong-named assemblies are in use, and up-to-date build tools are in place."  This has a lot of the functionality I had described in my post <a href="http://0xdabbad00.com/2010/09/12/idea-tool-to-rate-use-of-defense-in-depth/">Idea: Tool to rate use of defense-in-depth</a>.  

When I tried to install this tool originally, I got the error "Error 1001. Object reference not set to an instance of an object".  To bypass this, uncheck the option to integrate with Visual Studio.  I was running Windows XP x64, and anything less than Vista is supposedly unsupported.

The tool is a little buggy (I had to click back and forth between buttons for it to realize my input field wasn't actually empty for example), and it's entirely GUI based.  I tried running it on a few binaries on my system (notepad++, an old version of IDA Pro, and Foxit reader) and really all it seems to be able to do is 3 checks:
<ul>
<li>Is DEP enabled? Which the tool refers to as it's NXCheck.
<li>Is ASLR enabled? Which the tool refers to as it's DBCheck.
<li>Is SEH checking enabled? Which the tool refers to as it's SafeSEHCheck.
</ul>
So really, the tool isn't that cool, and confusing, as it's terminology is weird.
It has a bunch of other checks, but they all require the .pdb file with debug info, so that's not helpful for checking random vendor binaries.

<b>AppVerifier</b>
I couldn't get this to really do anything useful.  The basic idea is you select the program you want to test, and select a bunch of checks for it, then you run the program under windbg, and it's supposed to give you a list of issues it identified, but in my tests, either it found nothing, or it wouldn't let the program even run depending on what checks I had enabled.  So not useful.

<b>Attack Surface Analyzer BETA</b>
Won't let me install on Windows XP x64, only supports Vista and up.  The idea behind it is it "takes a snapshot of your system state before and after the installation of product(s) and displays the changes to a number of key elements of the Windows attack surface."  Because I can't install it, it is not useful.

<b>Conclusion</b>
After an hour of trying these tools out and watching the videos for them and docs for them, I have to conclude that they are not useful.  The concepts are pretty good, but the implementation is lacking.  They need to fix their bugs, and support more OS's.  I'm guessing the whole point of these tools is to allow some consultants to more easily generate some pretty XML to show managers when these Microsoft employees go to Adobe or whoever and try to get them to fix up their code.

