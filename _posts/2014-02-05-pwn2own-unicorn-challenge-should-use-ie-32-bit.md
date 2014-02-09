---
layout: post
title: Pwn2Own Unicorn challenge should use IE 32-bit
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1391743382'
  _edit_last: '2'
---
<b>Update</b> As <a href="https://twitter.com/fjserna">Fermin J Serna</a> <a href="https://twitter.com/fjserna/status/431481466682228736">pointed out</a>, 64-bit IE is harder to exploit.  My goal in this post was to show that EMET has diminishing returns for targets that are already well protected, so if the goal of the challenge is to exercise EMET, then 32-bit IE is a better target.  Also, as <a href="https://twitter.com/jness">Jonathan Ness</a> <a href="https://twitter.com/jness/status/431479946830024704">pointed out</a>, 64-bit IE does still use 32-bit processes for tabs on Windows 8.1 x64, unless special configuration steps are taken by the user, so he is seeking clarification from ZDI as to whether the process actually will be 64-bit that is exploited or not.

This year's Pwn2Own prize at <a href="http://cansecwest.com/">CanSecWest</a> features a Grand Prize of $150K for a "<a href="http://threatpost.com/pwn2own-paying-150000-grand-prize-for-microsoft-emet-bypass/104015">Unicorn Exploit</a>" that works on the following:

<blockquote>SYSTEM-level code execution on Windows 8.1 x64 on Internet Explorer 11 x64 with EMET (Enhanced Mitigation Experience Toolkit) bypass 
</blockquote>

Another <a href="http://www.pwn2own.com/2014/01/pwn2owns-new-exploit-unicorn-prize-additional-background-civilians/">post</a> on the site explains "The third and ultimate test for our contestants is to break through EMET protections and truly control the computer."

EMET is awesome, and I highly recommend everyone on Windows use it, but you get the most value out of EMET when you use it to secure an older OS running older software.  When you use a more modern environment, you already get a lot of protections without EMET.  Using the Pwn2Own Unicorn challenge as an example, here are the protections EMET offers and which of those you already have without EMET.

<table><tr>
<th>EMET Protection</th><th>Added security?</th><th>Explanation</th></tr>
<tr><td>DEP<td bgcolor="#FF8080">No<td>64-bit processes have DEP enabled by default, and IE is compiled for DEP as well.</tr>
<tr><td>SEHOP<td bgcolor="#FF8080">No<td>SEHOP is not relevant to 64-bit processes.</tr>
<tr><td>ASLR<td bgcolor="#FF8080">No<td>IE 11 already has ASLR enabled.</tr>
<tr><td>Certificate Pinning<td bgcolor="#FF8080">No<td>According to the EMET User Guide (page 17, table 6), Certificate Pinning does not work in Windows 8.  I don't have a system to test with to confirm, but this wouldn't be a relevant protection for this competition anyway.</tr>
<tr><td>Null page<td bgcolor="#FF8080">No<td>Windows 8 already protects the Null page</tr>
<tr><td>Heap spray<td bgcolor="#99FF99"><b>Yes</b><td>EMET will alloc 14 specific pages which are commonly used in heap sprays.  However, these are commonly used in 32-bit heap sprays, not 64-bit.  I'd also like to say heap sprays aren't as useful in 64-bit processes, but Ivan Fratric <a href="http://ifsec.blogspot.com/2013/11/exploiting-internet-explorer-11-64-bit.html">proved that wrong</a>, and he did this for IE 11 64-bit on Win 8.1 (the same set-up this prize is for)! Someone buy that man a beer.</td></tr>
<tr><td>EAF<td bgcolor="#99FF99"><b>Yes</b><td>Works, but there are a couple of known ways of bypassing, see my paper <a href="http://0xdabbad00.com/wp-content/uploads/2013/11/emet_4_1_uncovered.pdf">EMET 4.1 Uncovered</a></tr>
<tr><td>Anti-Rop<ul>
<li>LoadLib
<li>MemProt
<li>Caller
<li>SimExecFlow
<li>StackPivot
</ul><td bgcolor="#FF8080">No<td>EMET does not apply any of the Anti-Rop protections to 64-bit processes.</td></tr>
</table>

<h3>Conclusion</h3>
The only protections EMET is adding for this challenge are the Heap Spray and EAF protections, which are by far the easiest of the protections to bypass.  So the additional $50K for this challenge, over the $100K browser bounty for IE 11, is really for getting SYSTEM-level execution.  This challenge would actually be more difficult if this was for IE 11 <u>32-bit</u>, because then you'd have to circumvent the Anti-ROP protections from EMET.  There are known ways of circumventing these as well, but it's at least a little more challenging.

<i>Greetz to <a href="https://twitter.com/InsanityBit">@InsanityBit</a> for pointing this out</i>
