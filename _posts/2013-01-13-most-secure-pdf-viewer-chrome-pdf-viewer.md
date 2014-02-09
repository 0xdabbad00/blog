---
layout: post
title: ! 'Most Secure PDF viewer: Chrome PDF Viewer'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1358115664'
  _edit_last: '2'
---
Stop using Adobe Reader and Foxit Reader.  <b>Use Google Chrome's PDF viewer.</b>  I didn't do any tests or research to make this determination.   Just my gut feeling.

Although Adobe has improved the security situation a lot with sand-boxing Reader and allowing you to disable javascript, Adobe Reader is still a bloated monstrosity (it can play flash files!) and a large attack surface like that is hard to secure.

Foxit is a Chinese company, and my jingoist box isn't letting anything with that origin near it. :) Actually, there recently was a <a href="http://www.h-online.com/security/news/item/Current-Foxit-Reader-can-execute-malicious-code-1780636.html">vulnerability for Foxit Reader</a> which was a trivial buffer overflow of a filename that is too long, which is such a basic security check, that for me, it really calls into question the rest of the security of the software.  

Foxit Reader is not sandboxed in any way.  Adobe Reader's sandbox only restricts it's privileges.  Google Chrome has become the standard in how to make a proper sandbox on Windows with it's <a href="https://en.wikipedia.org/wiki/Google_Native_Client">NaCL technology</a>.

If when you look at a PDF in Google Chrome, it has the following set of icons, then you're using Google Chrome's PDF viewer.
<a href="http://0xdabbad00.com/wp-content/uploads/2013/01/chrome_1060734_buttons.png"><img src="http://0xdabbad00.com/wp-content/uploads/2013/01/chrome_1060734_buttons.png" alt="" title="Chrome PDF Viewer buttons" width="257" height="33" class="aligncenter size-full wp-image-790" /></a>
Double-check by going to <tt>chrome://plugins/</tt>.  I only have "Chrome PDF Viewer" and "Adobe Flash Player" enabled there.

This viewer meets all my needs.  You can drag and drop files to Chrome and it will open them automatically with the PDF Viewer.  I have uninstalled all other viewers.  There are various posts online saying the technology underlying this is either Adobe's or Foxit's, but I think it's <a href="https://sites.google.com/site/skiadocs/user-documentation/pdf-theory-of-operation">Google's Skia engine</a>.  In any case, I feel most comfortable with this as my PDF viewer.

Another interesting alternative (if you're the type that really likes to avoid the crowds to stay secure), is to use <a href="http://mozilla.github.com/pdf.js/web/viewer.html">PDF.js</a> which is entirely Javascript alternative, and has a Firefox plug-in. <b>Update 2013-01-13</b>: Looks like Mozilla will soon use this as their default PDF viewer according to <a href="https://blog.mozilla.org/futurereleases/2013/01/11/mozilla-tests-a-built-in-secure-pdf-viewer-in-firefox-beta-leveraging-the-power-of-html5/">blog.mozilla.org</a>.

Using <a href="https://live.gnome.org/Evince/Downloads">evince</a>, <a href="http://pages.cs.wisc.edu/~ghost/">GSview</a>, <a href="http://gnuwin32.sourceforge.net/packages/xpdf.htm">xpdf</a>, and some of the other alternatives are just not great user experiences in my opinion, and any time you try using a "Built for Linux" solution in Windows, things tend to not be as secure (they are often compiled with GCC and end up not having DEP and ASLR enabled).
