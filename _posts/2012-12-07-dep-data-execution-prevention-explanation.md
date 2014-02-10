---
layout: post
title: DEP (Data Execution Prevention) explanation by example
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1357492235'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
There is some confusion about DEP and the need for it.  People seem to incorrectly think that Java and .NET apps and other things that do JIT can't use DEP.

Let's walk through some examples of code to show exactly what DEP is protecting.  This may be surprising for those that think Java can't use DEP.  Here is our sample code:

{% highlight c %}
#include <windows.h>

int main(int argc, char **argv) {
	char shellcode[] = {
		0x66, 0x81, 0xE4, 0xFC, 0xFF, 0x31, 0xF6, 0x56, 0x64, 0x8B, 0x76, 0x30, 0x8B, 0x76, 0x0C, 0x8B,
		0x76, 0x1C, 0x8B, 0x6E, 0x08, 0x8B, 0x36, 0x8B, 0x5D, 0x3C, 0x8B, 0x5C, 0x1D, 0x78, 0x01, 0xEB,
		0x8B, 0x4B, 0x18, 0x67, 0xE3, 0xEC, 0x8B, 0x7B, 0x20, 0x01, 0xEF, 0x8B, 0x7C, 0x8F, 0xFC, 0x01,
		0xEF, 0x31, 0xC0, 0x99, 0x32, 0x17, 0x66, 0xC1, 0xCA, 0x01, 0xAE, 0x75, 0xF7, 0x66, 0x81, 0xFA,
		0x2A, 0xB6, 0x74, 0x09, 0x66, 0x81, 0xFA, 0xAA, 0x1A, 0xE0, 0xDB, 0x75, 0xC5, 0x8B, 0x53, 0x24,
		0x01, 0xEA, 0x0F, 0xB7, 0x14, 0x4A, 0x8B, 0x7B, 0x1C, 0x01, 0xEF, 0x03, 0x2C, 0x97, 0x85, 0xF6,
		0x74, 0x15, 0x68, 0x33, 0x32, 0x20, 0x20, 0x68, 0x75, 0x73, 0x65, 0x72, 0x54, 0xFF, 0xD5, 0x95,
		0x31, 0xF6, 0xE9, 0xA0, 0xFF, 0xFF, 0xFF, 0x56, 0x68, 0x72, 0x6C, 0x64, 0x21, 0x68, 0x6F, 0x20,
		0x77, 0x6F, 0x68, 0x48, 0x65, 0x6C, 0x6C, 0x54, 0x87, 0x04, 0x24, 0x50, 0x50, 0x56, 0xFF, 0xD5,
		0xCC
	};

	void *exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	memcpy(exec, shellcode, sizeof shellcode);
	((void(*)())exec)();
}
{% endhighlight %}

I took this shellcode from <a href="http://code.google.com/p/w32-msgbox-shellcode/">http://code.google.com/p/w32-msgbox-shellcode/</a>.  It simply pops up a message box saying "Hello world" and then the final 0xCC (an `int 3`) at the end causes a break-point.  Line 17 allocates memory that has all permissions (read, write, execute).   Then I copy the shellcode to that location, and line 19 runs it as if it were a function (it looks weird but that line works).

<b>This shellcode will execute whether DEP is enabled or not.</b>  I specifically requested execute privileges on the memory I alloc'd, so DEP does not protect against that. I asked for it.  That is why JIT works.  If you specifically say "I want this to execute", then DEP will not stop it from executing.

If you'd like to try this out, then in Visual Studio look for the /NXCOMPAT option under Project->Properties->Linker->Advanced.  "/NXCOMPAT:NO" means no DEP (no protection), whereas the default "/NXCOMPAT" means "Yes" protect the executable with DEP.

<a href="http://0xdabbad00.com/wp-content/uploads/2012/12/DEP_VS2010.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/12/DEP_VS2010-300x211.png" alt="" title="VS2010 DEP setting" width="300" height="211" class="alignnone size-medium wp-image-621" /></a>

When run, this causes the following pop-ups.

<a href="http://0xdabbad00.com/wp-content/uploads/2012/12/messagebox_shellcode.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/12/messagebox_shellcode.png" alt="" title="Shellcode" width="144" height="144" class="alignnone size-full wp-image-622" /></a> <a href="http://0xdabbad00.com/wp-content/uploads/2012/12/breakpoint.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/12/breakpoint-300x131.png" alt="" title="breakpoint" width="300" height="131" class="alignnone size-medium wp-image-623" /></a>

The second pop-up may vary (I was running while debugging under Visual Studio).

Now let's change the VirtualAlloc line to:
{% highlight c %}
void *exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT, PAGE_READWRITE);
{% endhighlight %}
This time things do not work because I am requesting `PAGE_READWRITE` instead of `PAGE_EXECUTE_READWRITE`.  <b>If you do not specifically request execute privileges, and you compile with DEP, DEP will not let that memory execute.</b>  This time I get this message:

<a href="http://0xdabbad00.com/wp-content/uploads/2012/12/access_violation.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/12/access_violation-300x131.png" alt="" title="Access Violation" width="300" height="131" class="alignnone size-medium wp-image-626" /></a>

The error code 0xC0000005 means "Access Violation" meaning that I tried to execute memory that was not executable.  Thank you DEP!  Now if you turn off DEP and recompile, it pops the "Hello world" message again.

So this seems good so far, but most code does not specify the permissions on the memory when it alloc's it.  Instead you just do:
{% highlight c %}
void *exec = (void *)LocalAlloc(LMEM_ZEROINIT, sizeof shellcode);
{% endhighlight %}

When you use `LocalAlloc` (or any other memory function where you don't specify the permissions), it's basically the same as specifying as calling `VirtualAlloc` with `PAGE_READWRITE`, because with DEP you are protected, but without, the shellcode runs.

Now JIT code does not just alloc all it's mem `PAGE_EXECUTE_READWRITE`.  You should never have memory that is both writable and executable.  That's asking for trouble.  Instead you should alloc the mem with read/write perms, copy the data in, change the permissions to execute only (read is not actually necessary), then execute it:
{% highlight c %}
void *exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT, PAGE_READWRITE);
memcpy(exec, shellcode, sizeof shellcode);
DWORD ignore;
VirtualProtect(exec, sizeof shellcode, PAGE_EXECUTE, &amp;ignore);
((void(*)())exec)();
{% endhighlight %}
This code works with DEP enabled.

<h3>What about 64-bit?</h3>
<b>All 64-bit executables are DEP enabled, no matter what.</b>  For simplicity (since I'm too lazy to find any 64-bit shellcode), let's use this shellcode in our same program:

{% highlight c %}
	char shellcode[] = { 
		0xCC
	};
{% endhighlight %}

A 32-bit exe that copied this to memory that was not specified as being executable, and was compiled without DEP protection, would pop a message about a break point.  With DEP it would pop a message about an access violation.  In 64-bit, whether or not you enable DEP (I checked the NXCOMPAT flag in the binary using <a href="http://icebuddha.com/">http://icebuddha.com/</a>), it will always pop a message about an access violation.  If you change the permissions to `PAGE_EXECUTE_READWRITE` it will pop a message about a break point.

<h3>What about DLL's?</h3>
<b>Only the .exe matters for DEP.</b>   To test this, let's make a DLL (I call mine test_dll.dll) with our original code:
{% highlight c %}
#include <windows.h>

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) {
    char shellcode[] = {
        0x66, 0x81, 0xE4, 0xFC, 0xFF, 0x31, 0xF6, 0x56, 0x64, 0x8B, 0x76, 0x30, 0x8B, 0x76, 0x0C, 0x8B,
        0x76, 0x1C, 0x8B, 0x6E, 0x08, 0x8B, 0x36, 0x8B, 0x5D, 0x3C, 0x8B, 0x5C, 0x1D, 0x78, 0x01, 0xEB,
        0x8B, 0x4B, 0x18, 0x67, 0xE3, 0xEC, 0x8B, 0x7B, 0x20, 0x01, 0xEF, 0x8B, 0x7C, 0x8F, 0xFC, 0x01,
        0xEF, 0x31, 0xC0, 0x99, 0x32, 0x17, 0x66, 0xC1, 0xCA, 0x01, 0xAE, 0x75, 0xF7, 0x66, 0x81, 0xFA,
        0x2A, 0xB6, 0x74, 0x09, 0x66, 0x81, 0xFA, 0xAA, 0x1A, 0xE0, 0xDB, 0x75, 0xC5, 0x8B, 0x53, 0x24,
        0x01, 0xEA, 0x0F, 0xB7, 0x14, 0x4A, 0x8B, 0x7B, 0x1C, 0x01, 0xEF, 0x03, 0x2C, 0x97, 0x85, 0xF6,
        0x74, 0x15, 0x68, 0x33, 0x32, 0x20, 0x20, 0x68, 0x75, 0x73, 0x65, 0x72, 0x54, 0xFF, 0xD5, 0x95,
        0x31, 0xF6, 0xE9, 0xA0, 0xFF, 0xFF, 0xFF, 0x56, 0x68, 0x72, 0x6C, 0x64, 0x21, 0x68, 0x6F, 0x20,
        0x77, 0x6F, 0x68, 0x48, 0x65, 0x6C, 0x6C, 0x54, 0x87, 0x04, 0x24, 0x50, 0x50, 0x56, 0xFF, 0xD5,
        0xCC
    };

    void *exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT, PAGE_READWRITE);
    memcpy(exec, shellcode, sizeof shellcode);
    ((void(*)())exec)();
}
{% endhighlight %}

Then for the .exe just use:
{% highlight c %}
#include <windows.h>

int main(int argc, char **argv) {
	LoadLibrary("test_dll.dll");
}
{% endhighlight %}

Please do not code like this in the real world.  I am just showing examples using the bare minimum of code.

If you disable DEP in the .exe, then you will get the "Hello world" message, no matter what protections are set on the DLL.  If you enable DEP on the .exe, then you will get an access violation, no matter what protections are set on the DLL.

A simple truth table to reinforce the point:
<table border=0 cellpadding=0 cellspacing=0>
<tr><th>EXE has DEP enabled<th>DLL has DEP enabled<th>Process is fully DEP protected</tr>
<tr><td style="background:#81F781">Yes<td style="background:#81F781">Yes<td style="background:#81F781">Yes
<tr><td style="background:#81F781">Yes<td style="background:#F5A9A9">No<td style="background:#81F781">Yes
<tr><td style="background:#F5A9A9">No<td style="background:#81F781">Yes<td style="background:#F5A9A9">No
<tr><td style="background:#F5A9A9">No<td style="background:#F5A9A9">No<td style="background:#F5A9A9">No
</table>

Although it does not matter if DEP is enabled for a DLL, I believe that not seeing the NXCOMPAT flag set in the DLL is weird, and indicates a weird build system.

<h3>What about drivers</h3>
Windows .sys files are PE files and thus can have the NXCOMPAT flag just like .exe files and any other PE file.  However, I'm pretty sure DEP doesn't matter for drivers. Any kernel developers please pass any knowledge you might have here to me.  Microsoft drivers do not have this flag set.

<h3>Does process X have DEP enabled?</h3>
This is tricky. The best way to figure this out is to use <a href="http://technet.microsoft.com/en-us/sysinternals/bb896653.aspx">Process Explorer</a>.  There are a couple of ways DEP can be enabled beyond having the NXCOMPAT flag set in the binary (which is all my tool <a href="http://icebuddha.com/slopfinder.htm">SlopFinder</a> checks for), and even if that is set, other Windows settings may turn it off.

Doing some research, I put together this graph of most of the relevant things that are necessary for DEP to be enabled.
<a href="http://0xdabbad00.com/wp-content/uploads/2012/12/DEP.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/12/DEP-300x227.png" alt="" title="Determine if DEP is enabled" width="300" height="227" class="alignnone size-medium wp-image-612" /></a>

Again, use <a href="http://www.microsoft.com/en-us/download/details.aspx?id=30424">EMET</a> to enable DEP and other protections on Windows.  Then you can just ignore that chart.
