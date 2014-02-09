---
layout: post
title: Windows Hardening Guide
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1364406703'
  _edit_last: '2'
---
Collaborative post by <a href="https://twitter.com/0xdabbad00">@0xdabbad00</a> (<a href="http://0xdabbad00.com/">0xdabbad00.com</a>) and <a href="https://twitter.com/InsanityBit">@insanitybit</a> (<a href="http://www.insanitybit.com/2013/03/27/windows-hardening-guide/">insanitybit.com</a>)

<h3>Audience</h3>
This guide is focused on Windows Vista, 7 and 8 systems for personal use.  This guide is not concerned with the following:

<ul>
<li>Not Windows XP or earlier because they simply do not have the security features necessary to securely use.  A lack of ASLR and SEHOP, no integrity levels, a kernel with exposed attack surface, and a general lack of privilege separation makes securing XP a task best left to science fiction.
<li>Not enterprise environments, though some of this information can certainly translate over
<li>No IDS, DNS log monitoring, or other network related activities that are usually only reasonable to spend time on in enterprise environments.
</ul>

<h3>Strategy</h3>
Disrupt, deny, and degrade attacks through reduction of attack surface area and implementation of modern mitigation techniques.  Finally, prepare for the worst, assume APT.
Reduce Attack Surface

Vulnerabilities require one thing - code; if the code exists, so will vulnerabilities. The best way to avoid being exploited is to ensure there as few vulnerabilities as possible for the attacker to exploit.  The simplest and most effective way to do that is to minimize the amount of software on the system - less running code means less places for your attacker to poke at.

There are some key areas that are commonly attacked:
<ol>
<li>PDF Reader:  If possible uninstall Adobe Reader and use Chrome or Firefox’s built in PDF reader.  If you must use Adobe Reader ensure that Javascript is disabled and that Protected Mode is enabled in the security settings. There will be other steps in the guide for hardening your PDF reader further.
<li>Java: Java is one of the most highly exploited programs on Windows systems. It’s a very easy target for attackers, and this is unlikely to change for a long time. If you can’t remove Java altogether I highly suggest changing your browser settings to “Click To Play Plugins”.
<li>Windows Services: Windows, like any other mainstream OS, comes with a ‘default compatible’ attitude - it has to work for everyone. That means it comes with a large number of services running by default. These services are exploitable, and <a href="https://www.pcworld.com/article/206010/Microsoft_Confirms_It_Missed_Stuxnet_Print_Spooler_Zero_Day.html">have been used for local privilege escalation in the past</a>. Disable any Windows services that you don’t need. Deciding which services you do or don’t need requires <a href="http://www.blackviper.com/windows-services/">a bit of research</a>, as different users require different things.
</ol>

For other software you’ve installed, such as an instant messaging client or torrenting client, always ensure that you have the latest version and keep track of security releases.  Many software applications have their own auto-update mechanisms, make sure you enable it if you don’t think you’ll stay on top of patching yourself.  You can also use software like <a href="https://secunia.com/vulnerability_scanning/personal/">Secunia’s PSI</a> which will scan the software you have installed to ensure it is up-to-date.  Secunia PSI can be useful to install once and check for out-of-date software, but it’s somewhat awkward to use and have running regularly, so I uninstall it after running it once. Alternatively you can use the FileHippo updater, which is portable and will check for any out of date software in its repository.

<h3>Disrupt Exploits</h3>
Given the possibility that your software may be vulnerable to 0-day threats or known threats that have not yet been patched, the next line of defense is to use techniques that disrupt exploits from being successful.  This is what EMET does.  It takes a bit of configuration, so use insanitybit’s write-up as a guide: <a href="http://www.insanitybit.com/2012/07/26/setting-up-emet-3-5-tech-preview-9-2/">http://www.insanitybit.com/2012/07/26/setting-up-emet-3-5-tech-preview-9-2/</a>

If an exploit does manage to get execution, the next line of defense is to break it’s ability to work correctly by denying it access to different APIs.  The best solution for this is <a href="http://ambuships.com/">AmbushIPS</a> by <a href="https://twitter.com/scriptjunkie1">@scriptjunkie1</a>.  This will protect best against ROP based exploits (which usually disable DEP as one of their steps which AmbushIPS check for), but also against exploits which have obtained full arbitrary execution.  If the attacker knows your are using AmbushIPS, he could likely modify his exploit to work around it, so to some degree this is security through obscurity, but setting up an IDS/IPS can prove very beneficial to those willing to manage them.  You can also write your own signatures for AmbushIPS to check for, which adds further unknowns for attacks.

AmbushIPS cannot only block exploits, but it can also log chosen Windows API calls to a remote server.  This could be helpful in identifying when an attack occurred and how, post-mortem.

<h3>Block Payloads</h3>
Although the stage in which an attacker launches their payload is both optional and late in the game, those looking to improve their security may look into AppLocker, an Anti-Executable security solution available for the more enterprise oriented Windows editions (Windows Server 2008 R2, Windows 7 Ultimate and Enterprise, Windows Server 2012, and Windows 8 Enterprise). Anti-Executable software works by preventing processes from launching based on a whitelist and blacklist. If Firefox.exe is running, and it tries to run evil.exe, and evil.exe is not whitelisted, then it will not run. This is most helpful for preventing malware that uses legacy techniques, and making it more difficult for an attacker to gain persistence.

AppLocker rules come in three types: path, hash, and my favorite, publisher.

A path rule is really quite weak. It basically says that ‘only files from this path can execute’, which means that all an attacker has to do to bypass that rule is write to the path and execute.

Hash rules are much more difficult to get around, but they’re also horribly difficult to maintain. Every time your program updates you need a new hash.

Publisher rules are based on certificate information. This is much easier to deal with, as it’ll only allow specific programs to run, but it won’t have to be updated for every program update.

While AppLocker is not enough for any attack that accounts for it, it can be useful when layered on top of other techniques. Just be sure that you realize its shortcomings.

<h3>Prepare For The Worst</h3>
Given the possibility that your laptop could just simply be stolen, encrypt your data with TrueCrypt (free) or Windows BitLocker (if you have Windows Enterprise or Ultimate editions).  Any and all sensitive information (ex. proprietary code for your company if you are a software developer) should generally be stored in some type of encrypted container.  Be aware that if you try to only encrypt specific data, Windows will still save a hibernation file (a copy of the RAM) to the system partition which may contain your sensitive information.

Here are guides for <a href="http://www.insanitybit.com/2012/07/20/setting-up-a-truecrypt-file-container-11/">TrueCrypt</a> and <a href="http://www.insanitybit.com/2013/01/08/bitlocker-drive-encryption-guide/">BitLocker</a>.

<h3>Security advice not specific to Windows</h3>
Your browser is your main attack surface on a personal system, so take efforts to secure that by using various extensions (NoScript and HTTPS-Everywhere). You can find guides for securing Firefox and Chrome <a href="http://www.insanitybit.com/2012/06/02/the-definitive-guide-for-securing-firefox/">here</a> and <a href="http://www.insanitybit.com/2012/06/02/the-definitive-guide-for-securing-chrome/">here</a>. As a user if you secure your browser you’re securing the area that most attackers will attempt to exploit.

Many websites now offer dual-factor authentication, such as GMail and Facebook.  Take advantage of these, so you don’t end up getting locked out of your own email and social network sites if you ever get owned.

Do your banking from a different computer that you use infrequently, but still keep up-to-date on patches!  Have your various website accounts send password resets to an email account that you only access from this banking computer. Make sure you’re connecting to these websites through a secure and trusted network.

<h3>Conclusion</h3>
There is a lot of security software for Windows out there: Some legitimately adds protection, and some unfortunately exposes you to more attacks than it protects you from.  It’s impossible to cover it all in a single post, so we tried to stick to the built-in and free tools that are most important.

If you follow this guide you’ll be making an attackers job much more difficult. Though there is no silver bullet, and Windows security software is somewhat limited, you can use this guide to significantly improve your chances when facing the latest 0-day exploit in your browser.

As always, if you have suggestions for the guide, corrections, or general comments, please feel free to leave that all in the comments section and we’ll have a look. <i>(Comment on <a href="http://www.insanitybit.com/2013/03/27/windows-hardening-guide/">insanitybit.com</a>)</i>
