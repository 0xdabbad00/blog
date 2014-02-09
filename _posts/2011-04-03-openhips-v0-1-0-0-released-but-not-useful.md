---
layout: post
title: OpenHIPS v0.1.0.0 released... but not useful
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1301861956'
  _edit_last: '2'
---
This release of OpenHIPS includes <a href="http://code.google.com/p/yara-project/">YARA</a> support for scanning for <a href="http://www.metasploit.com/">metasploit</a> shellcode.  However, this doesn't do a good job of protecting against metasploit, as I'll explain.

In this release, I stripped out most of the HeapLocker functionality that was in here previously.  I left in the check to see how much memory is being used which can potentially detect heap sprays (it suspends all threads in the process if the process's memory usage exceeds 400MB).  I also left in the code that scans memory pages, but instead of using a single <a href="http://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm">KMP</a> search for the string "unpack" like HeapLocker does, I'm using YARA, and I'm scanning for 3 shell code patterns that are used by metasploit (the payloads for shell_bind_tcp, shell_bind_tcp_xpfw, and shell_reverse_tcp).  Unfortunately, I can't get these patterns to hit, so this release of OpenHIPS isn't going to secure you.

The problem with writing a HIPS like this, is you don't know what to scan and when.  HeapLocker and currently OpenHIPS work by iterating through all of the memory allocations in a process repeatedly.  In the first run through when the projects first get execution, they mark all the memory as "scanned" without actually scanning it, because if they scanned it, then they would get false positives because the memory containing the signatures to look for would get scanned, which would alert on the signatures.  Also, you assume that the initial app doesn't have "shellcode" or whatever in it, so you don't scan it for efficiency and again to reduce the possibility of false positives.  However, what this means is that if I run one of the msfpayload executables that just executes the payload without exploiting anything, then OpenHIPS won't detect the shellcode, even though it's in the binary plain as day, but it's not in new memory allocations which is where OpenHIPS looks.

So ok, OpenHIPS only look at new memory allocations, but then we run into a race condition, because memory might get alloc'd and then scanned, before the shell-code is written to that memory page.  My choices, as a developer for a HIPS, are then to either repeatedly re-scan memory, which will have a performance impact, or scan memory under certain conditions, such as by hooking certain function calls.  I want to have generic solutions, so I want to only hook Windows API functions, and not application specific functions that would require I do some reverse engineering.  I could hook the Windows API call's for free'ing memory, which should have the shellcode in it (unless the shellcode clean's itself up by overwriting itself), but this means I have to wait for the shellcode to execute and do it's damage before I detect it, which I don't like.  I could hook calls to VirtualProtectEx which will hit whenever the protections on the memory changes, which should detect attacks against JIT compiled code, like Java and .Net, but memory created by javascript will probably never call this.

Finally, I will run into problems if the shellcode is encoded or obfuscated.  Yuck.

Anyway, expect a new release in a week or two that hopefully does a better job of catching metasploit.

<b>OpenHIPS v0.1.0.0</b>
<img src="http://0xdabbad00.com/wp-content/uploads/2011/03/openhips_logo.png" alt="OpenHIPS logo" /> <a href="https://www.assembla.com/code/openhips/subversion/nodes">Source code</a> | <a href="https://www.assembla.com/code/openhips/subversion/nodes/Releases/v0.1.0.0/openhips-setup.msi?_format=raw&amp;rev=124">Installer</a> | <a href="https://www.assembla.com/code/openhips/subversion/nodes/Releases/v0.1.0.0/openhips-setup_debug.msi?_format=raw&amp;rev=124">Debug Installer</a>
