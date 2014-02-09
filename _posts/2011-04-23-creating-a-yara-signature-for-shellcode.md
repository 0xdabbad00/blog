---
layout: post
title: Creating a YARA signature for shellcode
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1303749256'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
In working on <a href="https://www.assembla.com/wiki/show/openhips/">OpenHIPS</a>, I need to generate <a href="http://code.google.com/p/yara-project/">YARA</a> signatures for shellcode to detect the exploits used against browsers and client applications, like Adobe Reader.  The first thing I did was use <a href="http://www.metasploit.com/">metasploit</a> to generate a "malicious" PDF.  Starting up metasploit 3 on Windows, I tried a couple of exploits before I came across one that worked well, as shown:
[sourcecode language="text"]
msf &gt; use exploit/windows/fileformat/adobe_geticon
msf exploit(adobe_geticon) &gt; set PAYLOAD windows/messagebox
PAYLOAD =&gt; windows/messagebox
msf exploit(adobe_geticon) &gt; exploit

[*] Creating 'msf.pdf' file...
[*] Generated output file C:/metasploit/msf3/data/exploits/msf.pdf
[*] Exploit completed, but no session was created.
[/sourcecode]

This exploit uses a heap spray, so I commented out the code in OpenHIPS to detect large memory usage in my testing.  Next I brought up a Windows XP SP2 VM and installed Adobe 8.1.2 from <a href="http://www.oldapps.com/adobe_reader.php">oldapps.com</a>.  I opened up the <tt>msf.pdf</tt> file with this version of Adobe and after a while (it takes a while to allocate about 700MB for the heap spray), it popped up a little message box.

With the exploited Adobe Reader running, I attached WinDbg to it, and typed in the following in order to find the thread that had spawned the messagebox:
[sourcecode language="text"]
&gt; ~* k
0  Id: 100.54c Suspend: 1 Teb: 7ffdf000 Unfrozen
ChildEBP RetAddr  
0012d8cc 77d493f5 ntdll!KiFastSystemCallRet
0012d904 77d5688a USER32!NtUserWaitMessage+0xc
0012d92c 77d6b7c5 USER32!InternalDialogBox+0xd0
0012dbec 77d6b12b USER32!SoftModalMessageBox+0x938
0012dd3c 77d95fdf USER32!MessageBoxWorker+0x2ba
0012dd94 77d96084 USER32!MessageBoxTimeoutW+0x7a
0012ddc8 77d80598 USER32!MessageBoxTimeoutA+0x9c
0012dde8 77d80550 USER32!MessageBoxExA+0x1b
0012de04 0a11fff2 USER32!MessageBoxA+0x45
WARNING: Frame IP not in any known module. Following frames may be wrong.
...

&gt; u 0a11fff2
[/sourcecode]

The command <tt>~* k</tt> shows a stack backtrace of all the functions for every thread.  I cut out all the other threads.  Then I looked at the assembly code at address <tt>0x0a11fff2</tt> where the <tt>MessageBoxA</tt> call was made from.  I scrolled back up through the assembly instructions, and luckily I knew what I was looking for, because metasploit is open-source, so I can look at the file <tt>C:\metasploit\msf3\modules\payloads\singles\windows\messagebox.rb</tt> and I found the following assembly inside the ruby code interesting because it is searching through memory for the kernel32.dll file, and then looking for a specific function (MessageBoxA).  This looks like a good signature to me.
[sourcecode language="text"]
;get kernel32
	xor ecx,ecx
	mov esi, [fs:ecx + 0x30]
	mov esi, [esi + 0x0C]
	mov esi, [esi + 0x1C]
next_module:
	mov eax, [esi + 0x08]
	mov edi, [esi + 0x20]
	mov esi, [esi]
	cmp [edi + 12*2], cl
	jne next_module

	pop ecx
	add ecx,edx
	jmp ecx            ;jmp start_main

find_function:
	pushad				;save all registers
	mov ebp, [esp  +  0x24]	;put base address of module that is being loaded in ebp
	mov eax, [ebp  +  0x3c]	;skip over MSDOS header
	mov edx, [ebp  +  eax  +  0x78]	;go to export table and put relative address in edx
	add edx, ebp			;add base address to it.
						;edx = absolute address of export table
	mov ecx, [edx  +  0x18]		;set up counter ECX
						;(how many exported items are in array ?)
	mov ebx, [edx  +  0x20]		;put names table relative offset in ebx
	add ebx, ebp			;add base address to it.
						;ebx = absolute address of names table

find_function_loop:
	jecxz  find_function_finished ;if ecx=0, then last symbol has been checked.
						;(should never happen)
						;unless function could not be found
	dec ecx				;ecx=ecx-1
	mov esi,  [ebx  +  ecx  *  4]	;get relative offset of the name associated
						;with the current symbol
						;and store offset in esi
	add esi,  ebp			;add base address.
						;esi = absolute address of current symbol

compute_hash:
	xor edi,  edi			;zero out edi
	xor eax,  eax			;zero out eax
	cld					;clear direction flag.
						;will make sure that it increments instead of
						;decrements when using lods*

compute_hash_again:
	lodsb					;load bytes at esi (current symbol name)
						;into al, + increment esi
	test al, al				;bitwise test :
						;see if end of string has been reached
	jz  compute_hash_finished	;if zero flag is set = end of string reached
	ror edi,  0xd			;if zero flag is not set, rotate current
						;value of hash 13 bits to the right
	add edi, eax			;add current character of symbol name
						;to hash accumulator
	jmp compute_hash_again		;continue loop

compute_hash_finished:

find_function_compare:
	cmp edi,  [esp  +  0x28]	;see if computed hash matches requested hash
						; (at esp+0x28)
						;edi = current computed hash
						;esi = current function name (string)
	jnz find_function_loop		;no match, go to next symbol
	mov ebx,  [edx  +  0x24]	;if match : extract ordinals table
						;relative offset and put in ebx
	add ebx,  ebp			;add base address.
						;ebx = absolute address of ordinals address table
	mov cx,  [ebx  +  2  *  ecx]	;get current symbol ordinal number (2 bytes)
	mov ebx,  [edx  +  0x1c]	;get address table relative and put in ebx
	add ebx,  ebp			;add base address.
						;ebx = absolute address of address table
	mov eax,  [ebx  +  4  *  ecx]	;get relative function offset from its ordinal
						;and put in eax
	add eax,  ebp			;add base address.
						;eax = absolute address of function address
	mov [esp  +  0x1c],  eax	;overwrite stack copy of eax so popad
						;will return function address in eax
find_function_finished:
	popad 				;restore original registers.
						;eax will contain function address
	ret
[/sourcecode]

Correlating this back to the actual memory in the program, I can see the following:
[sourcecode language="text"]
0a11fef5 31c9            xor     ecx,ecx
0a11fef7 648b7130        mov     esi,dword ptr fs:[ecx+30h]
0a11fefb 8b760c          mov     esi,dword ptr [esi+0Ch]
0a11fefe 8b761c          mov     esi,dword ptr [esi+1Ch]
0a11ff01 8b4608          mov     eax,dword ptr [esi+8]
0a11ff04 8b7e20          mov     edi,dword ptr [esi+20h]
0a11ff07 8b36            mov     esi,dword ptr [esi]
0a11ff09 384f18          cmp     byte ptr [edi+18h],cl
0a11ff0c 75f3            jne     0a11ff01
0a11ff0e 59              pop     ecx
0a11ff0f 01d1            add     ecx,edx
0a11ff11 ffe1            jmp     ecx
0a11ff13 60              pushad
0a11ff14 8b6c2424        mov     ebp,dword ptr [esp+24h]
0a11ff18 8b453c          mov     eax,dword ptr [ebp+3Ch]
0a11ff1b 8b542878        mov     edx,dword ptr [eax+ebp+78h]
0a11ff1f 01ea            add     edx,ebp
0a11ff21 8b4a18          mov     ecx,dword ptr [edx+18h]
0a11ff24 8b5a20          mov     ebx,dword ptr [edx+20h]
0a11ff27 01eb            add     ebx,ebp
0a11ff29 e334            jecxz   0a11ff5f
0a11ff2b 49              dec     ecx
0a11ff2c 8b348b          mov     esi,dword ptr [ebx+ecx*4]
0a11ff2f 01ee            add     esi,ebp
0a11ff31 31ff            xor     edi,edi
0a11ff33 31c0            xor     eax,eax
0a11ff35 fc              cld
0a11ff36 ac              lods    byte ptr [esi]
0a11ff37 84c0            test    al,al
0a11ff39 7407            je      0a11ff42
0a11ff3b c1cf0d          ror     edi,0Dh
0a11ff3e 01c7            add     edi,eax
0a11ff40 ebf4            jmp     0a11ff36
0a11ff42 3b7c2428        cmp     edi,dword ptr [esp+28h]
0a11ff46 75e1            jne     0a11ff29
0a11ff48 8b5a24          mov     ebx,dword ptr [edx+24h]
0a11ff4b 01eb            add     ebx,ebp
0a11ff4d 668b0c4b        mov     cx,word ptr [ebx+ecx*2]
0a11ff51 8b5a1c          mov     ebx,dword ptr [edx+1Ch]
0a11ff54 01eb            add     ebx,ebp
0a11ff56 8b048b          mov     eax,dword ptr [ebx+ecx*4]
0a11ff59 01e8            add     eax,ebp
0a11ff5b 8944241c        mov     dword ptr [esp+1Ch],eax
0a11ff5f 61              popad
0a11ff60 c3              ret
[/sourcecode]

You always want to use the actual memory signature.  I copied this to a file, and ran the following commands against it in cygwin to get a nice YARA signature:
[sourcecode language="bash"]
$ awk 'BEGIN {FS=&quot; &quot;} {printf &quot;%s&quot;, $2}' tmp.txt | sed 's/\([0-9a-f]\{2\}\)/\1
/g'
31 c9 64 8b 71 30 8b 76 0c 8b 76 1c 8b 46 08 8b 7e 20 8b 36 38 4f 18
 75 f3 59 01 d1 ff e1 60 8b 6c 24 24 8b 45 3c 8b 54 28 78 01 ea 8b 4a 18 8b 5a 2
0 01 eb e3 34 49 8b 34 8b 01 ee 31 ff 31 c0 fc ac 84 c0 74 07 c1 cf 0d 01 c7 eb
f4 3b 7c 24 28 75 e1 8b 5a 24 01 eb 66 8b 0c 4b 8b 5a 1c 01 eb 8b 04 8b 01 e8 89
 44 24 1c 61 c3
[/sourcecode]

This then ends up in my rules.yar file as:
[sourcecode language="text"]
rule shellcodeFindKernel32
{
	meta:
		sourceOrg = &quot;metasploit&quot;
		sourcePath = &quot;msf3\\modules\\payloads\\singles\\windows\\messagebox.rb&quot;
	strings:
		$string = {31 c9 64 8b 71 30 8b 76 0c 8b 76 1c 8b 46 08 8b 7e 20 8b 36 38 4f 18 75 f3 59 01 d1 ff e1 60 8b 6c 24 24 8b 45 3c 8b 54 28 78 01 ea 8b 4a 18 8b 5a 20 01 eb e3 34 49 8b 34 8b 01 ee 31 ff 31 c0 fc ac 84 c0 74 07 c1 cf 0d 01 c7 eb f4 3b 7c 24 28 75 e1 8b 5a 24 01 eb 66 8b 0c 4b 8b 5a 1c 01 eb 8b 04 8b 01 e8 89 44 24 1c 61 c3}
	condition: $string
}
[/sourcecode]

The next release of OpenHIPS will detect this shellcode, although it will detect the heap spray using up a lot of memory first, or hopefully the instructions used in any nop sled in a heap spray.
