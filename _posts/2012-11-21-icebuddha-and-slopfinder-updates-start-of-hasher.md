---
layout: post
title: IceBuddha and SlopFinder updates + start of Hasher
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1353532505'
  _edit_last: '2'
---
I've been continuing to work on <a href="http://icebuddha.com">IceBuddha</a>.
<ul>
<li>Now correctly parses the first few structures in a 32-bit or 64-bit Windows executable.
<li>Has "goto" functionality so you can specify a byte location to go to.  It is actually also an eval (with a little regex ontop of it) so you can specify a location as "100h + 10" = 0x10A.
<li>Has "colorize" feature to show all the parts of a struct. Access by right-clicking on the parse tree.
<a href="http://0xdabbad00.com/wp-content/uploads/2012/11/icebuddha1.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/11/icebuddha1-300x290.png" alt="" title="Colorize" width="300" height="290" class="alignnone size-medium wp-image-534" /></a>
<li>Includes <a href="http://ace.ajax.org/#nav=about">ACE editor</a> (used by the <a href="https://c9.io/">Cloud9 editor</a>).  This allows you to edit the parse script and have it's changes applied when you click back to the parse tree immediately.  Note that because this whole app is javascript and html with no server interaction, your changes will not be saved.  Hopefully that will be possible in the future.
<a href="http://0xdabbad00.com/wp-content/uploads/2012/11/icebuddha2.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/11/icebuddha2-300x290.png" alt="" title="Ace editor" width="300" height="290" class="alignnone size-medium wp-image-536" /></a>
<li>Searches the binary for strings and allows you to go to them when you click on them.
<a href="http://0xdabbad00.com/wp-content/uploads/2012/11/icebuddha3.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/11/icebuddha3-300x290.png" alt="" title="Strings" width="300" height="290" class="alignnone size-medium wp-image-537" /></a>
</ul>

<a href="http://icebuddha.com/slopfinder.htm">SlopFinder</a> has also been updated to be a little prettier and work with both 32-bit and 64-bit binaries.  I think at this point, it's safe to say you should try it out against whatever binaries you want and go yell at the developers if it is a commercial company, or just politely request or send a code diff if it is an open-source project.
<a href="http://0xdabbad00.com/wp-content/uploads/2012/11/slopfinder3.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/11/slopfinder3-300x290.png" alt="" title="SlopFinder" width="300" height="290" class="alignnone size-medium wp-image-540" /></a>

I have also started working on <a href="http://icebuddha.com/hasher.htm">Hasher</a>, which is a quick and dirty tool to let you drag and drop files and see their hash.  The idea came about when someone at my office was asking if there was a tool to do this (and one that was trusted enough for the company's customers).  I knew it would be easy (at least to have minimal functionality for his use case), so I hacked one together.  It does not support files over 10MB though, and it hashes all the files in their entirety in memory at the same time.
<a href="http://0xdabbad00.com/wp-content/uploads/2012/11/Hasher.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/11/Hasher-300x290.png" alt="" title="Hasher" width="300" height="290" class="alignnone size-medium wp-image-542" /></a>

You can also use <a href="http://onlinemd5.com/">http://onlinemd5.com/</a> for doing what Hasher does, but that site sketches me out because there is no contact info, no "About", nothing.  Creepy.  At least have an alias like I do. :)

I have a plan to create some tools to help people doing support, and eventually try to "port" a lot of the bash command-line tools to the browser, but I'm going to start with just seeing what I can do with web versions made for the web (so GUI versions) and trying to see what I ultimately want to do with that functionality.  For example, maybe support wants to tell someone to grep a log inside a tar ball for any errors so it could all be done in the browser.  Just a thought, probably one of those things that only sounds cool for a very limited set of problems, so not worth the effort.
