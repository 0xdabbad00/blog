---
layout: post
title: Document formatting
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1384924172'
  _edit_last: '2'
---
Some people commented they liked the look of my <a href="http://0xdabbad00.com/2013/11/18/emet-4-1-uncovered/">EMET report</a>, so I've uploaded the <a href="https://github.com/0xdabbad00/research/tree/master/emet_uncovered">.tex file on github</a>.  

I originally tried writing the whole report in <a href="http://www.codinghorror.com/blog/2012/10/the-future-of-markdown.html">markdown</a> and using <a href="http://johnmacfarlane.net/pandoc/">pandoc</a> to convert it to a PDF, as I had plans of releasing it as both a PDF and html.  Unfortunately, I hit the limits of what could be done with pandoc and ended up with a "buggy" PDF.  So I used pandoc to output a .tex file.

My latex code is hacky, but I like the output'd PDF.  If you want to write your own report, I highly recommend using <a href="http://www.xm1math.net/texmaker/">Texmaker</a>.  It will automatically download any packages mentioned in the .tex file for you so you don't have to fight dependencies on your own.  If you've never used LaTeX before, you'll "Build" your PDF, then run "bibtex emet" to convert the emet.bib file to something that can be used by LaTeX.  Then hit "Build" twice more, and you'll end up with a PDF with the proper reference numbers.  It sounds kludgy, but that's how it's done.
