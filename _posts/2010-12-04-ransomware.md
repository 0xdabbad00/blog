---
layout: post
title: Ransomware
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _edit_lock: '1294279820'
---
Kaspersky has <a href="http://www.securelist.com/en/blog/208188032/
And_Now_an_MBR_Ransomware">some</a> <a href="http://www.securelist.com/en/blog/333/GpCode_like_Ransomware_Is_Back">articles</a> on some ransomeware, which is basically malware that encrypts some files on your computer, or in some way denies you access to your computer unless you pay them some money. Historically ransomware used weak crypto, but now it's strong.  What interests me though is that ransomware provides a way for criminals to profit from their malware (unlike viruses thrown about randomly) without it needing to call back home to them (which programs that steal banking credentials need to do).  So most A/V heuristics are going to look for code that looks odd (based on whatever that is) and that calls home.  Ransomeware doesn't need to call home though.  The user "calls home" for them. 
