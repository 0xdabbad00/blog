---
layout: post
title: Binary compilation differences
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1390759925'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
<h3>Summary</h3>
I compiled the same code with a bunch of different compilers and have made those binaries available.  I also verified that the putty.exe 0.63 that can be downloaded matches what can be built from source code for it.  <b>This post is just for my archive, nothing cool is introduced and I do not identify what any of the differences are between compilers.</b>

<h3>Introduction</h3>
Given the post a while back on <a href="https://madiba.encs.concordia.ca/~x_decarn/truecrypt-binaries-analysis/">binary verification of the TrueCrypt binaries</a>, I decided to match some binaries myself and I was interested in what differences do exist between different versions of compilers.

<h3>Compiler output</h3>
I decided to compile the same code with different compilers to see what the differences would be.  I've made those binaries available at: 
<b><a href='http://0xdabbad00.com/wp-content/uploads/2014/01/compilations_with_different_compilers.zip'>compilations\_with\_different\_compilers.zip</a></b> (2.82MB)

The included README.txt explains much of what was done.  I compiled very simple binaries as my primary interest was on what the PE headers are that are generated.  The projects used the following source codes:
{% highlight c %}
int main(int argc, char** argv) {
  return 42;
}
{% endhighlight %}
<p>
{% highlight c %}
int factorial(int a) {
  if (a == 1) return a;
  return a * factorial(a - 1);
}

int main(int argc, char** argv) {
  return factorial(argc);
}
{% endhighlight %}

I compiled debug and release builds where possible with the following compilers using all defaults settings:
<ol>
<li><a href="http://openwatcom.mirrors.pair.com/open-watcom-c-dos-1.9.exe">Watcom 1.9 DOS</a>
<li><a href="http://forms.embarcadero.com/forms/BCC32CompilerDownload">Borland C++ Compiler 5.5.1</a>
<li><a href="http://www.bloodshed.net/dev/devcpp.html">Dev-C++ 5 beta release</a> (4.9.9.2)
<li>Intel Composer XE Evaluation 2013 SP1
<li><a href="http://www.cs.virginia.edu/~lcc-win32/">lcc-win32</a> 3.8
<li><a href="http://www.mingw.org/">MinGW</a> using gcc, cpp, and g++ which all appear to be links to gcc 4.8.1
<li><a href="http://www.cygwin.com/">Cygwin</a> using gcc, cpp, and g++ which all appear to be links to gcc 4.8.2
<li>Visual Studio 6
<li>Visual Studio 2003 Pro
<li>Visual Studio 2003 Pro SP1
<li>Visual Studio 2005 Pro
<li>Visual C++ 2008 Express
<li>Visual Studio 2010 Express
<li>Visual Studio 2012 Express
<li>Visual Studio 2013 Express
</ol>

For each project there are 35 binaries.  I also have a few builds of putty in there.  Builds were done on Windows XP SP3 and Windows 7.

If you want to write signatures for compilations like <a href="http://hbgary.com/free_tools">HBGary's Fingerprint</a> tool once sort of did to try to group malware, you should use <a href="https://www.hex-rays.com/products/ida/tech/flirt/index.shtml">IDA FLIRT signatures</a>.  These binaries are just there for those interested as they did take a stupid amount of time to generate (download and install the compiler and figure out how to use it in some cases).

<h3>Putty 0.63 binary compilation verified</h3>
I verified that you can compile the source code for putty.exe 0.63 to obtain the same binary.  This confirms there is no trojaned functionality in that binary outside of whatever might be in the source code, which I did not review.

<h4>Procedure</h4>
<ol>
<li>Install "Visual Studio .NET 2003 Professional - Full Install (English)" from the MSDN on a vanilla Windows XP SP3 system.  Do not download or install any updates to Visual Studio.
<li>Download the <a href="http://the.earth.li/~sgtatham/putty/latest/putty-src.zip">putty source</a>.  (MD5: 21dadf391eed109dd89c1befe96cac88)
<li>Download the <a href="http://tartarus.org/~simon/putty-snapshots/x86/putty.zip">putty binaries</a>.  (MD5: 2af64c860af7af67a25d639e2fcba006)
<li>These MD5's are listed on putty's page <a href="http://the.earth.li/~sgtatham/putty/0.63/md5sums">here</a>.  Ensure you are getting the release code and binaries, not the development versions.
<li>Extract the files
<li>Modify putty-src/WINDOWS/PUTTY.MFT to change the line endings from '\r\n' to '\n'. I don't know why this was this was this way. This can be done with cygwin via: <tt>sed -i 's/^M$//' PUTTY.MFT</tt>
<li>Bring up the Visual Studio command prompt, cd to putty-src/WINDOWS, and run <tt>nmake -f
 MAKEFILE.VC VER=/DRELEASE=0.63</tt>
<li>Compare your new putty.exe against the downloaded putty.exe and you should see only a 3 or 4 byte difference at 0x100 for the link time.
</ol>
