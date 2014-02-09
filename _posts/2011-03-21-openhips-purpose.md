---
layout: post
title: OpenHIPS purpose
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1300756560'
  _edit_last: '2'
---
OpenHIPS is an open source Windows Host Intrusion Prevention System focused on client systems.  I believe there is an opportunity for this software, and to explain why, let's first go through some of attack avenues facing computers today, which will help explain why OpenHIPS is needed and what it will do.  

<b>Attacks faced by users today</b>
Most home computers today have a network firewall of some sort.  Either the Windows firewall, or the network device from their ISP will block incoming connections so you don't have to worry about things like the <a href="http://en.wikipedia.org/wiki/Blaster_%28computer_worm%29">Blaster</a> worm anymore directly connecting to services running on your computer.  Not many users install random software anymore, and when you do install software you get it from a somewhat reputable source (like from it's homepage), so most viruses and trojans aren't an issue anymore.

So home users are mostly concerned today with client side attacks, wherein they somehow browse to a site that serves malicious content, or they open a .pdf, .doc, or other file containing malicious content which exploits their software, giving the malicious content execution on your system.  Google search results warn you if you are going to a malicious site, and many email providers will scan emails for malicious content, but it may still get through.  Now if it is a file you download and open, your anti-virus software may flag it, but again some may get through.

Once you open this malicious content, it needs to somehow gain execution through an exploit.  Ideally, you keep up on your patches, and the vendors of the software you use patch known vulnerabilities, but usually there is some lag time there, and also you could always run into the problem of being exploited by a 0-day exploit, so we need to break that exploit from working.

<b>White-listing</b>
In some cases the exploit may just be a dumb feature of the software, such as .pdf's having the ability to directly execute DLL's, you will need to be able to white-list which files are allowed to run.  I hope to add some sort of ability like this to OpenHIPS eventually, possibly by providing an easy interface to <a href="http://technet.microsoft.com/en-us/windows/dd320283">AppLocker</a> which is a feature of Windows 7 that is based on the Software Restriction Policies (SRP) of prior Windows OS's.  I have not had any experience with AppLocker yet though. However, if file white-listing is implemented then this will also break the later stages of many exploitations wherein they drop a file and try to execute it.

<b>Heap spray detection</b>
Exploits are usually some type of memory manipulation such as buffer overflows, to get execution of memory that is controlled by the attacker.  EMET from Microsoft goes a long way in providing additional protection here, as it can force an application to use DEP, ALSR, and other security measures.  I like to think of exploits as sticks of dynamites.  The exploitation has two main parts, getting the <a href="http://en.wikipedia.org/wiki/Shellcode">shellcode</a> into memory and giving it execution.  This is like placing the stick of dynamite and then lighting the fuse.   DEP prevents execution, so in this metaphor, DEP would be like submerging the dynamite in water so a flame can't touch it.  ALSR randomizes the stack, so it would be like hiding the stick of dynamite, so the attacker doesn't know where the fuse is to light it.  One work-around the attacker could perform is to use a heap spray, wherein they basically are tossing thousands or millions of sticks of dynamite into memory, so when they blindly try to light something, they are more likely to hit a fuse.  This is usually coupled with a NOP sled, which can be thought of as making the fuse really long, so you can light the fuse at the end, or in the middle, or anywhere else, and once it's lit it will eventually reach the dynamite, which is the shell-code part of the exploit.  OpenHIPS (by it's inclusion of HeapLocker) helps defeat this by identifying that the memory usage is extraordinarily high and will stop the application (and exploit) from running and inform the user of the problem.

<a href="http://research.microsoft.com/en-us/projects/nozzle/">Nozzle</a> from Microsoft is another tool that watches memory allocations, and according to the <a href="http://channel9.msdn.com/Blogs/Peli/Heap-Spraying-Attack-Detection-with-Nozzle"> video</a> on it, Nozzle also determines if the allocated data could be executed (does it contain legitimate instructions or would an exception occur if it gained execution), and if there are jumps going to the same place.  However, there is no download available.

There is also a tool called STRIDE that tries to look for NOP sleds, but again no download is available, and you'll have to look for the paper "STRIDE: Polymorphic Sled Detection Through Instruction Sequence Analysis" to read more.

HeapLocker is an open-source tool, so it was easy to integrate.  I'd also like to integrate the concepts discussed by <a href="http://www.kryptoslogic.com/">Kryptos Logic Research</a> in their paper titled <a href="http://www.kryptoslogic.com/download/JIT_Mitigations.pdf">JIT spraying and mitigations</a> by Piotr Bania which identifies some good concepts that I'd like to use.

<b>Shell code virus scanner</b>
Another thing I want OpenHIPS to do ultimately is basically be a virus scanner for exploits in memory.  SNORT signatures can look for exploits occurring on the wire, but most people don't use SNORT, and SNORT is an IDS (it detects things) it doesn't prevent things (like an IPS).  I believe I can look through memory for these exploits, by possibly using SNORT signatures, as was discussed by Peter Silberman for his <a href="http://www.mandiant.com/products/research/mandiant_mindsniffer/">MindSniffer</a> tool, presented at Blackhat DC 09 where you can see the <a href="https://www.blackhat.com/presentations/bh-dc-09/Silberman/BlackHat-DC-09-Silberman-Snort-My-Memory-slides.pdf">slides</a> and <a href="https://www.blackhat.com/presentations/bh-dc-09/Silberman/BlackHat-DC-09-Silberman-Snort-My-Memory-Whitepaper.pdf">paper</a> on the Blackhat archives.  The difference between OpenHIPS and Mindsniffer is that Mindsniffer is meant just for forensic work, so you run it once during an incident response.  OpenHIPS would be run on every document loaded into memory, giving us more control over what needs to be scanned and when, which should help reduce the performance impact.  I'd also like the make signatures for public shell-code from <a href="http://www.metasploit.com/">Metasploit</a> and other sources.

<b>Comparison to other open source projects</b>
In order to finish this discussion, I will also point out the open-source security tools that have existed for Windows.  <a href="http://wehntrust.codeplex.com/">WehnTrust</a> was an open-source project developed in 2008 by Scape (security bad-ass extraordinaire, see <a href="http://uninformed.org/">uninformed.org</a>) which added ASLR and SEH protection, but was focused on Windows 2000, XP, and Server 2003, whereas I am focusing more on Windows Vista, 2008, and 7, which already provide ASLR more or less as a default feature. It is also no longer active.

<a href="http://sourceforge.net/projects/winpooch/">WinPooch</a> was an open-source HIPS focused on white-listing allowed exe's and only allowing certain functions to run.  However, it was discontinued in 2008, and only worked on 2000, XP, 2003, 32-bit only.

So all the other Windows open-source HIPS products are no longer active, and their purposes are somewhat replaced by default Windows functionality in the case of WehnTrust, or Windows functionality that exists but isn't on, in the case of WinPooch.  All the other heap spray protection tools are not available for download, in the case of STRIDE, Nozzle, and the work of Kryptos Logic, or are not easy to install, in the case of HeapLocker.  So this is the niche OpenHIPS is filling.
