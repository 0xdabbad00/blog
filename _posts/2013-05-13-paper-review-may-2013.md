---
layout: post
title: Paper review - May 2013
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1368468183'
  _edit_last: '2'
---
I'm going to start publishing summaries here of papers I read that are interesting, pointing out the high-lights I read.  I have a lot of catching up to do, and sometimes I don't find out about a paper until later, so I will be reviewing some older papers here as well, but I'll make sure to include dates.  Let me know if these summaries are interesting to you by contacting me at 0xdabbad00 on gmail and twitter!

<h3><a href="https://www.microsoft.com/en-us/download/details.aspx?id=26788">Mitigating Software Vulnerabilities</a></h3>
Microsoft paper, July 2011, Authors: Matt Miller, Tim Burrell, Michael Howard
<ul>
<li>Great tables showing mitigations (DEP, ASLR, SEHOP and others) available for different versions of Windows, Visual Studio compilers, and versions of IE and Office.  Shows when different technologies were introduced.
<li>Describes the economics of exploitation and how these mitigations drive up costs for an attacker.  Using 3 tactics:
<ul>
<li>Enforce invariants: "Invalidate an attacker‘s implicit assumptions" - DEP, SEHOP
<li>Create artificial diversity: ASLR
<li>Leverage knowledge deficits: /GS (stack cookies)
</ul>
<li>"There are no known exploits for stack-based vulnerabilities that have been capable of bypassing the combination of /GS, SEHOP, DEP, and ASLR."
<li>"No exploits have been observed in the wild that rely on corrupting heap metadata and target Windows Vista and beyond. " Note this statement is only relevant to "heap metadata" corruptions.
<li>Links to this great presentation (video and audio, but no slides, 45min) from Matt Miller <a href="http://technet.microsoft.com/en-us/security/dd285253.aspx">BlueHat v8: Mitigations Unplugged</a>
</ul>

<h3><a href="research.microsoft.com/pubs/64363/tr-2007-153.pdf‎">Low-level Software Security: Attacks and Defenses</a></h3>
Microsoft paper, November 2007, Author: Ulfar Erlingsson

Describes memory corruption attacks, but more importantly to me, describes defenses (and their performance impacts) which are:
<ol>
<li>"Checking Stack Canaries on Return Address" - This is /GS.  Discusses how this protection is not applied to all functions due to heuristics, to try to be performant, but this allowed for the <a href="http://blogs.msdn.com/sdl/archive/2007/04/26/
lessons-learned-from-the-animated-cursor-security-bug.aspx">ANI vulnerability</a>.
<li>"Moving function-local variables below stack buffers" - Compiler can rearrange variables on the stack so a buffer overflow will not over-write other variables.
<li>"Make data not be executable as machine code" - DEP
<li>"Enforcing control-flow integrity on code exe" - The concept here is that for things like C-structs that contain function pointers, to avoid having these over-written with arbitrary function addresses and subsequently executed, you can check these if you happen to know that they can only be one of some set of possible values.
<li>"Encrypting addresses in code and data pointers" - Even though a function pointer might be over-written, encrypt it so that the attacker doesn't know what value to over-write it with.  This concept is used on Vista's heap metadata.
<li>"Randomizing the layout of code and data in memory" - ASLR
</ol>


<h3><a href="http://www.acq.osd.mil/dsb/reports/ResilientMilitarySystems.CyberThreat.pdf">TASK FORCE REPORT: Resilient Military Systems and the Advanced Cyber Threat</a></h3>
Defense Science Board, January 2013, dozens of authors, approx 90 pages.
<ul>
<li>Categorizes adversaries into:
<ol>
<li>Those that can take advantage of known threats. 
<li>Those that can find 0-days
<li>Those that can create vulnerabilities in systems (Tier V-VI threats)
</ol>
<li>The only countries capable of creating vulnerabilities, according to the report, are the Russians, Chinese, and US.  Those that create vulnerabilities are basically those that can modify the supply chain or leverage insiders.  Willing to spend billions of dollars and years to do so.  Provides the example of <a href="http://www.nsa.gov/public_info/_files/cryptologic_histories/Learning_from_the_Enemy.pdf">The Gunman Project</a>.  According to the report, these advanced threats require the US to spy on adversaries in order to know about them at all.
<li>Section 8 is the most interesting part to read, as it discusses "Enhancing Defenses to Thwart Low- and Mid-Tier Threats", and provides a success story of how this has been accomplished in the Dept of State.  See pages 59-62, which describe:
<ul>
<li>"8.2.1.3 Automate Patch and Threat Management Functions" states "Over time, fewer staff should be needed to maintain software patches and network configurations, allowing a shift in effort toward hunting adversaries who have penetrated our networks. Most of the COTS technologies available today have user 
interfaces that allow high levels of flexibility for determining what is deemed unusual network behavior, allowing system administrators to adjust and adapt the monitoring systems as threats evolve."
<li>"8.2.1.4 Audit to the Enterprise Standard" - Discusses improvement of security posture not only in terms of technical solutions but also through "peer pressure" by grading personnel and holding managers responsible.
<li>"8.2.1.5 Build Network Recovery Capability" - Advocates having a back-up network and systems to use while kicking an adversary out of one network.
<li>"8.2.1.6 Recover to a Known (Trusted) State" - Have the ability to rapidly revert to back-ups.
</ul>
<li>Like much of the thought leadership occurring in cyber, it advocates better defined career paths for "cyber warriors".  You can tell this doc is gov focused, as it mentions "cyber" repeatedly (911 times!). 
<li>Strong focus in the report on nuclear.  Discusses how US nuclear defense should be a guide for how cyber could be similarly organized and implemented (isolated and very different than other capabilities).
</ul>
