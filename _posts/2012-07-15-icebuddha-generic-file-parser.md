---
layout: post
title: ! 'IceBuddha: Generic file parser'
categories: []
tags: []
status: publish
type: post
published: true
---
<h3>FileReader</h3>
I have been working sporadically on a project for the past few months to see how far I can take a webapp.  As indicated in my post <a href="http://0xdabbad00.com/2012/03/18/html5-filereader/">HTML5 FileReader</a>, I have been interested in using the HTML5 FileReader functionality to read local files.  Using this feature, I created <a href="http://icebuddha.com">IceBuddha.com</a>.  This project began as a web-based hex-viewer for binary files that can be drag and dropped to the page.  When you initially drop a file, the first 1600 bytes are read, and as you scroll down, more and more of the file is read in.  This allows you to drop 20MB file with no performance impact, but this is going to make other functionality difficult, so I might change this.   For now, thank you <a href="http://imakewebthings.com/jquery-waypoints/">http://imakewebthings.com/jquery-waypoints/</a> for making a lot of the functionality for that possible.



The cool part about using the FileReader functionality is everything is happening locally in javascript in your browser.  So once visit the site and you download a few 100K of javascript, you can drag and drop megs and megs of files to the site and you won't have to wait for them to upload to my public server and I won't have to pay for that band-width because that data is all staying local to your system.  This is a pretty cool concept... we'll see how far I can take it.

<h3>Hex dump</h3>
Next, the file is displayed as a standard hex-dump with the address for every 16 bytes displayed and an ascii representation.

<a href="http://0xdabbad00.com/wp-content/uploads/2012/07/icebuddha-hexdump.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/07/icebuddha-hexdump-300x113.png" alt="" title="IceBuddha - hex dump" width="300" height="113" class="alignnone size-medium wp-image-444" /></a>

I did some GUI work so as you move the mouse around the hexdump you'll see the corresponding ASCII character high-lighted, and in the top right corner you'll see different representations of the data displayed.

<h3>File parsing</h3>
The hard-work for this has been in turning this into a generic binary file parser.  This would be a web app that allows you to see the internal structures for various file formats.  So if you look at a .exe file, you could see the imports, and if you look at a .jpg, you could see the geo location info, amongst other information.  People have various uses for these tools.  Either they want to tweak the information (in which case, I will need to turn this into a hex editor), or they simply want to study it for whatever reasons. For example, it might help you analyze malformed PDF files (as Didier Stevens wanted to do with his binary template for 010 editor <a href="http://blog.didierstevens.com/programs/pdf-tools/">http://blog.didierstevens.com/programs/pdf-tools/</a>), or it might help you analyze the .lnk file behind stuxnet.

There are a few generic binary file parsers already.  For Windows, there is an application called <a href="http://www.sweetscape.com/010editor/">010 editor</a>.  For OSX, there is <a href="http://www.synalysis.net/">Synalyze It!</a>.  The benefit of my tool would be that it would be cross-platform, free, and hopefully easier for people to create and edit the template files for various file types.  The template files describe how to parse the files.  Ideally, I would create a wiki to allow people to edit the files.

<h3>Template format</h3>
If you want people to help write templates for your project, you need to make it easy for them.  I assume most binary files have C headers describing parts of their structure.  Example, for PE files (.exe, .dll, and .sys files), you can look in <a href="http://source.winehq.org/source/include/winnt.h">winnt.h</a> for parts of it's structure.

Additionally, I need some logic to describe things like parse `IMAGE_OPTIONAL_HEADER` in a PE file at the offset given by `e_lfanew` in the `IMAGE_DOS_HEADER`.  Originally, I had attempted using a <a href="http://en.wikipedia.org/wiki/Domain-specific_language">DSL</a>, like 010 editor, but no one wants to learn a DSL.  So then I considered trying to use a language and library with more transferable skills by using the format of Wireshark's dissector files which allow a variety of languages (C, Lua, Python), but this would be a big and awkward under-taking to make this usable by JavaScript (would need to leverage a Python interpreter for JavaScript), and the formatting still seemed awkward to me.  I might regret this, but I decided to use JavaScript for the logic of the parsing.

To tie together the C header formatted data structure representations with the JavaScript logic, I use <a href="http://pegjs.majda.cz/">PEG.js</a> which is a Parser Generator for JavaScript to parse the C header struct's on the data provided.  A template file for a PE file thus looks like this:

{% highlight javascript %}
typedef struct IMAGE_DOS_HEADER {
	WORD  e_magic;      /* 00: MZ Header signature */
	WORD  e_cblp;       /* 02: Bytes on last page of file */
	WORD  e_cp;         /* 04: Pages in file */
	WORD  e_crlc;       /* 06: Relocations */
	WORD  e_cparhdr;    /* 08: Size of header in paragraphs */
	WORD  e_minalloc;   /* 0a: Minimum extra paragraphs needed */
	WORD  e_maxalloc;   /* 0c: Maximum extra paragraphs needed */
	WORD  e_ss;         /* 0e: Initial (relative) SS value */
	WORD  e_sp;         /* 10: Initial SP value */
	WORD  e_csum;       /* 12: Checksum */
	WORD  e_ip;         /* 14: Initial IP value */
	WORD  e_cs;         /* 16: Initial (relative) CS value */
	WORD  e_lfarlc;     /* 18: File address of relocation table */
	WORD  e_ovno;       /* 1a: Overlay number */
	WORD  e_res[4];     /* 1c: Reserved words */
	WORD  e_oemid;      /* 24: OEM identifier (for e_oeminfo) */
	WORD  e_oeminfo;    /* 26: OEM information; e_oemid specific */
	WORD  e_res2[10];   /* 28: Reserved words */
	DWORD e_lfanew;     /* 3c: Offset to extended header */
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;

typedef struct IMAGE_OPTIONAL_HEADER {
	WORD  Magic;
	BYTE  MajorLinkerVersion;
	BYTE  MinorLinkerVersion;
	DWORD SizeOfCode;
	DWORD SizeOfInitializedData;
	DWORD SizeOfUninitializedData;
	DWORD AddressOfEntryPoint;            /* 0x10 */
	DWORD BaseOfCode;
	DWORD BaseOfData;
	DWORD ImageBase;
	DWORD SectionAlignment;               /* 0x20 */
	DWORD FileAlignment;
	WORD  MajorOperatingSystemVersion;
	WORD  MinorOperatingSystemVersion;
	WORD  MajorImageVersion;
	WORD  MinorImageVersion;
	WORD  MajorSubsystemVersion;          /* 0x30 */
	WORD  MinorSubsystemVersion;
	DWORD Win32VersionValue;
	DWORD SizeOfImage;
	DWORD SizeOfHeaders;
	DWORD CheckSum;                       /* 0x40 */
	WORD  Subsystem;
	WORD  DllCharacteristics;
	DWORD SizeOfStackReserve;
	DWORD SizeOfStackCommit;
	DWORD SizeOfHeapReserve;              /* 0x50 */
	DWORD SizeOfHeapCommit;
	DWORD LoaderFlags;
	DWORD NumberOfRvaAndSizes;
	IMAGE_DATA_DIRECTORY DataDirectory[16]; /* 0x60 */
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;


var imageDosHeader = parseStruct(0, IMAGE_DOS_HEADER);
var e_lfanew = getStructValue(imageDosHeader, "e_lfanew");
parseStruct(e_lfanew, IMAGE_OPTIONAL_HEADER);
{% endhighlight %}

I want to essentially run an eval on this JavaScript file to parse the data given, and have it leverage a library of functions such as parseStruct() and getStructValue().  However, those structs make this totally broken JavaScript, so what I do is run regex's on this file to convert those structs to strings:
{% highlight javascript %}
var IMAGE_DOS_HEADER = "
typedef struct IMAGE_DOS_HEADER {
	WORD  e_magic;      /* 00: MZ Header signature */
	WORD  e_cblp;       /* 02: Bytes on last page of file */
	...
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
";
{% endhighlight %}

Next, JavaScript does not allow multi-line strings unless you end each line with a back-slash, but this apparently causes some browser issues, so my hack was simply to use a regex to convert the entire JavaScript file to a single line, and then eval that.  So far this hack has worked brilliantly for me!

<h3>Parsing Example</h3>
When a file is parsed, a tree of data is created that you can click through to see what data exists for various structures in the file.  The item you select will automatically be scrolled to and high-lighted.

<a href="http://0xdabbad00.com/wp-content/uploads/2012/07/IceBuddha_parsing.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/07/IceBuddha_parsing-300x214.png" alt="" title="IceBuddha Parsing" width="300" height="214" class="alignnone size-medium wp-image-455" /></a>

<h3>Project Status</h3>
This project is still in it's early stages, where a lot of things might change drastically, such as the template format.  However, I think it shows off a lot of cool functionality of things that can be done with the FileReader, and incorporating in a lot of JavaScript projects to make a very unique webapp.

There are a lot of cool things that I think could be done with this project.  I'm still not convinced making a webapp for this is a smart idea because I have already run into a fair number of performance hits (like trying to read in even 100KB file all at once) and many frustrations as I am new to JavaScript development and have been caught off-guard by a few things (like not easily being able to read other files on the web from JavaScript).

Here are some ideas I think would be cool for this:
<ul>
<li>More file templates.  Only a minimal PE parse is possible now.
<li>Automatic detection of file types.  Currently any file you drop there will be parsed as a PE file.
<li>More error handling.  Right now, I'm just adding features.  My error handling and reporting is horrible/non-existent right now.
<li>Back-end integration. Right now, the project is entirely .html, .css, and .js files.  You can download and run the whole site in an isolated environment if you want, which might be a big selling point for the project, but it would also be good to leverage a back-end server for something.  For example, this might make a cool front-end for developing YARA signatures, and then you could have your signature you create scanned against 100,000 known good files to make sure you aren't accidentally going to cause a false positive.  Or maybe it could upload some of the parsed data in order to find similar files.
<li>File diff'ing.  Binary file diff'ers are awkward, and usually only useful if you know what the actual data is that is different.
<li>Export parsed data.  This would be quick and easy to provide a download link for the tree of parse data that would download as a XML file.
<li>Various GUI work, such as providing a "colorize" feature that you could apply to different structures so that in the hexdump window you'd have all the bytes in rainbows of colors to make it easier to understand what bytes are responsible for what.
<li>Wiki capability so others can easily create and edit templates for various file formats
<li>Command-line, URL POST arguments, and cookie settings/user-login so various functions could be applied or configuration settings used.
</ul>

Go check the site out at: <a href="http://icebuddha.com">icebuddha.com</a>
