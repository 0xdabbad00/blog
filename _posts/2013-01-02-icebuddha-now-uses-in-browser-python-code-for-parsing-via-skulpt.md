---
layout: post
title: IceBuddha now uses in-browser python code for parsing via skulpt
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1357163032'
  _edit_last: '2'
---
One thing I've hated about <a href="http://icebuddha.com/">IceBuddha</a> was that the file parsing code was done in Javascript.  Well, half Javascript and half C structs that I was parsing with <a href="http://pegjs.majda.cz/">PEG.js</a>.  You can see this frankenstein monster in my post <a href="http://0xdabbad00.com/2012/07/15/icebuddha-generic-file-parser/">IceBuddha generic file parser</a> or see the actual parsing script in the old file on github <a href="https://github.com/0xdabbad00/icebuddha/blob/3a08130335a50fb1da213a374fce36b8fe9cb9ae/parseFile_pe.txt">parseFile_pe.txt</a>.  It worked well while I was developing the rest of IceBuddha and the dual language thing I think was pretty cool as it made up for some of, what I believe are, deficiencies in Javascript. Specifically, the lack of multi-line strings.

Security folks don't use Javascript.  There are no node.js security tools or anything else in Javascript.  Most modern security tools for analysis are written in Python or allow you to use Python in them.  Why should IceBuddha be different?  I bit the bullet and decided to port my parsing code to python, but I still wanted everything happening in the browser on the client, so I'm leveraging <a href="http://www.skulpt.org/">skulpt</a>.

Skulpt is an insane project to write a Python to Javascript translator in Javascript.  What I end up doing is passing the binary data you drop on IceBuddha to the python code, which parses it, and then spits back the names, offsets, and other info about what it found which IceBuddha's Javascript then interprets.  Things are a little kludgy right now, which is partly due to my laziness, but also Skulpt is not a complete Python interpreter.  I think most of this awkwardness is abstracted away from the user though, as the code you might actually modify is pretty clean looking.  Check it out in the file on github <a href="https://github.com/0xdabbad00/icebuddha/blob/6eb291903bd5de001c590b0807f93804d23a757f/parseFile_pe.py">parseFile_pe.py</a>, or look go to my test page for IceBuddha at <a href="http://icebuddha.com/index.htm?test=1">http://icebuddha.com/index.htm?test=1</a> and clicking "Parse as: EXE".

At this point I think most of the major issues I was worried about are taken care of now in IceBuddha.  Now I just need to crank out some parse scripts for different files (and improve the current PE parsing), make it possible to edit the files, and do some other minor changes.  It's important to note that even with all these libraries, the site is still under 300KB (when files get compressed else it's about 700KB), and if you've already cached things like jquery, it'll be smaller.  Compare this to youtube.com which is over 500KB or facebook.com which is over 1MB, and best of all, you can always just host IceBuddha yourself locally.

I'm excited about the python code because it means that once a script is written for IceBuddha it could be used outside of IceBuddha.  For example, you might use the parse scripts to scan files to identify which files have specific features, modify features, extract the features, or possibly use it to drive fuzzing.  These are some of the ideas I have planned.

As a final note, the beauty of the in-browser python code, is that you can play around and do whatever you want on IceBuddha and the only thing you'll break is your own session.  None of the python code is executed on my server.  And as always, none of your data is sent to my server.
