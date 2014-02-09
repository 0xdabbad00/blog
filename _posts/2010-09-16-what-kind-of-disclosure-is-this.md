---
layout: post
title: What kind of disclosure is this?
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1284688413'
  _edit_last: '2'
---
I've said in my <a href="http://0xdabbad00.com/about/">about page</a> that I am opposed to the concept of "responsible disclosure".  The reason is that it takes work to find vulnerabilities, and my opinion is best described by Dino Dai Zovi's post <a href="http://trailofbits.com/2009/03/22/no-more-free-bugs/">No More Free Bugs</a>.  On June 17, 2010, VirusBlokAda published a <a href="http://www.f-secure.com/weblog/archives/new_rootkit_en.pdf">pdf</a> announcing their discovery of Stuxnet (one of the most interesting malwares of all time!).  They disclosed that it had a 0-day exploit related to how Windows processes .lnk files to spread via thumb-drives (<a href="http://www.cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2010-2568">CVE-2010-2568</a>).  Also it was signed by a legit cert from Realtek (later a variant came out signed by certs from JMicron) and it sought out Siemen's SCADA systems.  What interests me right now is it didn't contain just one 0-day, but four 0-days!  However, it hasn't been until September 14, 2010 (patch Tuesday) that we <a href="http://www.microsoft.com/technet/security/bulletin/ms10-061.mspx">found out about and got a patch for</a> one of the additional exploits, which is in the Windows Print Spooler.  Microsoft <a href="http://blogs.technet.com/b/msrc/archive/2010/09/13/september-2010-security-bulletin-release.aspx">credits Kaspersky Lab and Symantec</a> for independently finding and verifying the additional exploit.  Kaspersky has <a href="http://www.securelist.com/en/blog/2291/Myrtus_and_Guava_Episode_MS10_061">posted</a> that there are still two additional 0-days (for escalation of privileges)!

So lets review the important dates:
<ul>
<li>0-day: June 17, Stuxnet found and announcement of the .lnk vulnerability
<li>Day 46: August 2, Microsoft releases patch <a href="http://www.microsoft.com/technet/security/bulletin/MS10-046.mspx">MS10-046</a> for the .lnk vulnerability.
<li>Day 89: September 14, Microsoft releases patch <a href="http://www.microsoft.com/technet/security/bulletin/ms10-061.mspx">MS10-061</a> for the Print Spooler vuln and discloses the existence of the exploit.
<li>Day ?: Patch for privilege escalation vuln 1.
<li>Day ?: Patch for privilege escalation vuln 2.
</ul>

So we all went 89 days totally unprotected and uneducated about the print spooler vuln while it was running around wild and free in a known malware.  This is disturbing.  I imagine bad guys are getting samples of these malware and reverse engineering them just like the good guys are, and probably incorporating what they find into their own code.  If they haven't been, they probably will now!  Further, they're probably looking at Stuxnet as we speak, trying to find the two yet to be disclosed privilege escalation vulns.

So what do we call this kind of "disclosure"?  And is the malware itself the disclosure of the exploit, or the patch?  And do we still call an exploit 0-day on day 89 when it hasn't been disclosed in a paper but existed publicly as a sample that could have been reverse engineered?
