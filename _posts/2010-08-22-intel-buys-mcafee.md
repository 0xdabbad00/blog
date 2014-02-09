---
layout: post
title: Intel buys McAfee
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1282523061'
  _edit_last: '2'
---
On Thursday, Intel bought McAfee, but no one seems to know why.  Intel makes hardware, McAfee makes security software.  People speculate that Intel wants to sell security software with it's chips, on the premise that people need both.  But McAfee really only runs on Windows, and you have to know the OS anyway in order to sell the software, so that doesn't make much sense.

Perhaps Intel is hoping to have McAfee run at a lower level, like a security hypervisor.  Thus far, hypervisors have been focused on security through separation.  McAfee focuses mostly on just detection, with some protection capabilities via it's HIPS products, but no virtualization or separation that I know of, so I guess it's possible you could have your hypervisor doing detection and protection.  If this is the case, well, I feel sort of like Intel has given up on security, in the sense that as a hardware vendor, they should be able to enforce security and introduce mechanisms that can't be subverted from software.  Intel chips have 4 rings that code can run at, but Windows only uses 2 (kernel in ring-0 and user code in ring 3).  Hypervisors came along and offered the ability to run at another ring essentially, ring -1, so perhaps Intel is trying to ensure that this new ring actually gets used properly to ensure security.  Or perhaps Intel will offer some sort of other security service.  This link (<a href="http://www.docstoc.com/docs/34943161/McAfee-Product-Strategy">http://www.docstoc.com/docs/34943161/McAfee-Product-Strategy</a>) points out that McAfee acquired SafeBoot in 2007.  SafeBoot provides full disk encryption services, which is something that would run at a low-level.

I'm not really sure what Intel plans on doing with McAfee, but here are some things McAfee has (based on my assumption that every A/V company has these):
<ul>
<li>A deployment mechanism for updates and management via it's ePO product and just generally with it's virus updates.
<li>Reverse engineering knowledge (required for analyzing malware).
<li>A large repo of known goodware and badware.
<li>A process in place for ensuring updates don't brick boxes, although even recently McAfee has messed this up.  This process should be large numbers of machines in all sorts of configurations with all sorts of software installed, so you can push out an update to this large test network and ensure your updates work safely with all them. 
</ul>

Those I think are the most useful assets of any A/V company.  The software (A/V software, and even HIPS to some degree) is largely a commodity.  McAfee's biggest differentiators over other A/V companies are it's size and it's ePO product, which is increasingly being used as a framework for "plug-ins" from under vendors (like HBGary) so you can manage and secure large groups of computers from a centralized server.
