---
layout: post
title: The Future of Endpoint Threat Detection and Response
categories: []
tags: []
status: publish
type: post
published: true
---

The current direction of the infosec industry, with regards to malware prevention and detection, is using two very similar technologies (with horrible acronyms).  These are:
<ul>
<li><b>Advanced Threat Protection (ATP)</b> which includes products like FireEye, Palo Alto's Wild Fire, LastLine, and <a href="http://www.cuckoosandbox.org/">Cuckoo Sandbox</a>.  These "detonate" samples in "sandboxes" (virtual machines).
<li><b>Endpoint Threat Detection & Response (ETDR)</b>, which was coined by Gartner's Anton Chuvakin (<a href="http://blogs.gartner.com/anton-chuvakin/2013/07/26/named-endpoint-threat-detection-response/#comments">here</a>), and includes Mandiant's MIR, CarbonBlack, CounterTack's Sentinel, CrowdStrike's Falcon, and Immunity's <a href="http://immunityproducts.blogspot.com/2014/04/revamping-el-jefe.html">El Jefe</a>.  These can be further categorized:
    <ul>
    <li>Tools that do scans, such as Mandiant's MIR.  This tells you what exists at the time of the scan, which are "<b>stateful properties</b>".
    <li>Tools that do real-time monitoring, such as CarbonBlack, CounterTack, and El Jefe. These tell you what is happening as it happens (with perhaps a few minutes of delay.  These are referred to as "<b>events</b>".  These are the newer products.
    </ul>
</ul>


To explain why these product lines are similar, let's examine how they work.  

<h3>How ATP works</h3>
<ol>
<li>Acquire sample files: Pull them off the wire as they cross the network, extract them from email attachments, and maybe grab them from host systems.
<li>Automatically spin up a virtualized environment for these to run in, send the sample to it, and execute it or open the file.  In the case of Cuckoo, most people use VirtualBox, but VMWare, Qemu, and other solutions are possible.
<li><b>Monitor</b>: Record what the sample does for a few minutes, and send these results back to the server.
<li><b>Analyze</b>: Check these results using various signatures, such as "Did Adobe Reader spawn a new process?" (which indicates it was exploited), or "Did a mutex that starts with '_AVIRA_'  get created?" which indicates Zeus.  More examples of these types of signatures can be seen in Cuckoo's <a href="https://github.com/cuckoobox/community/tree/master/modules/signatures">community repository</a>.  Generate alerts if needed.
<li><b>UI</b>: Provide a nice way of reviewing the data collected.
</ol>
<br>

<h3>How ETDR works</h3>
<ol>
<li>Install an agent on all host systems.
<li><b>Monitor</b>: Record what happens on those hosts, such as what processes were created, and send these results back to a server.
<li><b>Analyze</b>: Check these results using various signatures.
<li><b>UI</b>: Provide a nice way of reviewing the data collected.
</ol>
<br>

<h3>Blended solution</h3>
The last 3 bullets in each of those lists are the same, as I believe they are based on similar technology.  Each category has pros and cons though, for example ETDR products can not do heavy monitoring because that would impact the performance of the hosts.  As such, some ETDR products are tied into ATP products in order to provide further analysis.  For example, El Jefe identifies new processes and can collect samples of those executables which can then be sent to Cuckoo Sandbox to explain what that executable does.

These products will continue to merge, due to the following pros and cons of each product.
<table>
<tr><th>ATP<th>ETDR
<tr>
    <td bgcolor="#FF8080">Blind to some ways malware can be delivered to hosts because they largely rely on monitoring network traffic.  They will not detect malware from HTTPS sites or physical media without agents running on the host, which brings them towards ETDR territory.
    <td bgcolor="#99FF99">Sees every sample the host tries to execute.
<tr>
    <td bgcolor="#FF8080">Samples may not run correctly in the virtualized environment.
    <td bgcolor="#99FF99">Sees how samples really run on a live host with real network connectivity.
<tr>
    <td bgcolor="#99FF99">Can be slow and thorough in the analysis by monitoring everything.
    <td bgcolor="#FF8080">Can not monitor as much because it can not affect the performance of the host.
<tr>
    <td bgcolor="#99FF99">Can identify malicious samples and prevent them from running on hosts before they cause damage.
    <td bgcolor="#FF8080">Will largely be reactive.  Detects incident and allows for more efficient incident response by identifying which hosts were affected by a sample.
</table>

<h3>Benefits</h3>
This new generation of products (in comparison to the AV and Personal Security Products of the previous generation) are differentied by their <b>centralized "brains" and enterprise focus</b>.

AV's biggest problem is that any malware author can easy download and install every AV product and test their samples against it.  If it gets caught, they make changes until it isn't caught, and then can send the sample off to targets while happily content in the knowledge that no AV product will catch it.  

With this new generation of products, the signatures are largely going to be unknown to the attackers.  These products, and the Threat Intelligence feeds they use, cost money.  An attacker doesn't know what signatures the target might be using.  When the attackers do get caught, they don't know why.  <b>That's why these products are more effective</b>.

Unfortunately, the home user and small business is being left out in the cold for advanced protection now.  Currently, these products require a security team, or at least one human, to review their alerts, which of course a small business doesn't have.  As these products mature and become more automated, the addressable market should expand.  They could also expand into these markets via managed service offerings (the security company monitors the data collected), as these companies easily benefit from economies of scale, and they become more powerful with the more data they have.

It's also important to point out that this next generation of products has a much <b>smaller attack surface</b>.  As opposed to trying to parse things to identify what they "are", these products simply monitor to see what they "do".  The ETDRs must have minimal functionality in order to be performant, and there is no benefit in exploiting the ATP's because those are virtualized, throw-away environments.  These are almost all new companies (at least for the ETDRs), with new code, and hopefully better secure development practices.  The AV industry has numerous bugs in their products, as shown most recently by Joxean Koret at SYSCAN 2014 in his presentation <a href="http://mincore.c9x.org/breaking_av_software.pdf">Breaking Antivirus Software</a>.  The cobbler's children have always been barefoot.

<h3>Weaknesses</h3>
<h4>ATP Weaknesses</h4>
The greatest weakness on the ATP side is that malware will recognize it is being executed in a monitored environment and will disable itself.  Or alternatively, the malware will be DRM'd to only run on the intended target.  These faults are well-known and have, for example, been described in the Black Hat US 2012 presentation <a href="https://media.blackhat.com/bh-us-12/Briefings/Song/BH_US_12_Song_Royal_Flowers_Automated_Slides.pdf">Flowers for Automated Malware Analysis</a> by Chengyu Song and Paul Royal.  FireEye even has a paper describing how most sandboxes are bypassed on their own site called <a href="http://www.fireeye.com/resources/pdfs/fireeye-hot-knives-through-butter.pdf">Hot Knives Through Butter: Evading File-based Sandboxes</a>.

<h4>ETDR Weaknesses</h4>
For the ETDRs, attackers will either maintain a presence entirely in memory to avoid being noticed, or try to bounce up to kernel space where the attacker will be on equal footing with the monitoring and can bypass whatever techniques are being used there.

Another option would be for the attacker to find ways to disable the products.  It is difficult for an admin to tell the difference between a machine that has been powered off, versus one that is no longer responding because an attacker has taken control of it.  These products may need to get tied into the firewalls so the firewall will disable any traffic from a host who's agent is not at least sending heart beats.  However, the attacker could instead neuter the product so it still sends heart beats, but it no longer monitors properly.

Because these products are monitoring and not blocking actions, and because they likely will cache and package up multiple results at once to send home, a smart attacker may be able to time his actions to race against this heart beat.  He will try to wait for the latest heart beat, then do some malicious actions which would generate alerts once the server analyzes them, but these actions are meant to deny that heart beat, and any future ones, from being sent.


<h3>Signatures</h3>
The signatures for these products are similar and many of these companies provide Threat Intelligence (TI) feeds which are basically the signatures that can be applied to competing products or to firewalls.  My next post will try to make sense of some of the standards available.
