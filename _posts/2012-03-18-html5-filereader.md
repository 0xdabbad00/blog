---
layout: post
title: HTML5 FileReader
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1332087926'
  _edit_last: '2'
---
I'm starting to look more into what fun things can be done with javascript and HTML5 as a browser-based application.  One new feature that interests me is the <a href="http://dev.w3.org/2006/webapi/FileAPI/#filereader-interface">FileReader API</a> which allows you to read a local file from the web app without uploading it to the server.  The user must still drag and drop or input the file in someway so it can be accessed, else it'd be a bad security hole, but the idea of a client-side app doing something with a 200MB file without having to upload it to a server makes this interesting.

Mostly it seems on the net people are doing things with images for this.  For example, if someone tried to upload a 3000x2000 pixel image to a site the web page could first reduce that image to 600x400 before it uploaded it, saving tons of bandwidth and time.

I also noticed that <a href="https://www.virustotal.com/">virustotal</a> uses this to sha-256 hash a file that you want scanned before it uploads it.  Virustotal hashes the file then compares the hash to it's database.  By doing this it doesn't need to actually upload the file, since it already has a copy of the file.

