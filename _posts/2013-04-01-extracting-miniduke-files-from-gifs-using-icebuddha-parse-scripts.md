---
layout: post
title: Extracting MiniDuke files from gifs using IceBuddha parse scripts
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _syntaxhighlighter_encoded: '1'
  _edit_lock: '1378057059'
  _edit_last: '2'
---
This post is a tutorial on how to use <a href="http://icebuddha.com/">IceBuddha</a> and it's parse scripts in performing some malware analysis on Mini Duke.

On February 27, 2013, Kaspersky Labs released a paper <a href="https://www.securelist.com/en/downloads/vlpdfs/themysteryofthepdf0-dayassemblermicrobackdoor.pdf">The MiniDuke Mystery: PDF 0-day Government Spy Assembler 0x29A Micro Backdoor</a>.  The paper states that the Mini Duke PDF exploit connects to artas[dot]org/engine/index.php (and some other domains) where it downloads a gif file that contains an encrypted backdoor.

I grabbed a copy of one of the files (bg_afvd.gif: md5sum 92a2c993b7a1849f11e8a95defacd2f7).  I put the file in a password protected zip, using the same password scheme as <a href="http://contagiodump.blogspot.com">http://contagiodump.blogspot.com</a>, and you can download it here: <a href="http://www.mediafire.com/?x9h4bkv9y1aalil">bg_afvd.gif.zip</a> (md5sum: 39664ffaf93d8961d1498de8a7c49807).  It's just a normal .gif file with some data appended to the end, so it can't infect you, but it's always good to take precautions.

IceBuddha has the ability to parse .gif files.  You can take a look at one by going to <a href="http://icebuddha.com/index.htm?test=sample_1.gif">http://icebuddha.com/index.htm?test=sample_1.gif</a>
<a href="http://0xdabbad00.com/wp-content/uploads/2013/04/sample.png"><img src="http://0xdabbad00.com/wp-content/uploads/2013/04/sample-300x238.png" alt="" title="Sample opened in IceBuddha" width="300" height="238" class="aligncenter size-medium wp-image-922" /></a>

This sample file is the same one used by <a href="http://www.matthewflickinger.com/lab/whatsinagif/bits_and_bytes.asp">http://www.matthewflickinger.com/lab/whatsinagif/bits_and_bytes.asp</a> which is an excellent explanation of the GIF file format.

When I look at bg_afvd.gif in IceBuddha though, I see there is a lot of data after the trailer.
<a href="http://0xdabbad00.com/wp-content/uploads/2013/04/miniduke_gif.png"><img src="http://0xdabbad00.com/wp-content/uploads/2013/04/miniduke_gif-300x238.png" alt="" title="IceBuddha with Mini Duke gif file" width="300" height="238" class="aligncenter size-medium wp-image-924" /></a>

At this point I want to extract out the data after the trailer.  I could do this manually, but given that there are a number of MiniDuke gif files, it would be best to do this programmatically.  IceBuddha's parse scripts are python code that runs in the browser (using <a href="http://0xdabbad00.com/2013/01/29/things-ive-learned-using-skulpt-for-in-browser-python-code/">skulpt</a>), but you can just as easily run them locally by cloning the github project <a href="https://github.com/0xdabbad00/icebuddha">https://github.com/0xdabbad00/icebuddha</a>

Once you have that, go to the parse_scripts directory, and from here you can run:
<pre>python fileparser.py -t gif ~/Downloads/bg_afvd.gif</pre>
This will generate the following output:
<pre>
[
  {
    "label": "GIF", 
    "size": 1696, 
    "data": "", 
    "offset": 0, 
    "interpretation": "", 
    "children": [
      {
        "label": "GIFHEADER", 
        "size": 6, 
        "data": "", 
        "offset": 0, 
        "interpretation": "", 
        "children": [
          {
            "label": "Signature[3]", 
            "size": 3, 
            "data": "", 
            "offset": 0, 
            "interpretation": "GIF", 
            "children": []
          },
&lt;snip&gt;
</pre>

You can get the same data by right-clicking on the parse data in IceBuddha and selecting "Download parsed data".

You could then run this on all the MiniDuke gif files to identify the location of the trailer.  I've written a script to find the trailer, extract the key, decrypt the pe file and write it to disk, which you can run as follows:
<pre>
python example_extractMiniDukeFile.py ~/Downloads/bg_afvd.gif
22790 bytes found at end of file
Extracted file written to: /home/user/Downloads/bg_afvd.gif.infected
</pre>

This <a href="https://github.com/0xdabbad00/icebuddha/blob/master/parse_scripts/example_extractMiniDukeFile.py">script</a> (part of the github repo) finds the trailer offset:
[sourcecode lang="python" gutter="false"]
# Check if GIFTrailer is at the end of the file
parsedData = fileparser.parseFile(filename, filetype)

trailer = fileparser.findElement(parsedData, &quot;GIFTrailer&quot;)
trailerOffset = trailer['offset']
[/sourcecode]

It then reads in the file, extracts the key, and decrypts the remainder of the file, which it writes to bg_afvd.gif.infected,  This file has the md5sum 297ef5bf99b5e4fd413f3755ba6aad79 which you can search for in virustotal and the <a href="https://www.virustotal.com/en/file/c60621e82f58b5ea5b36cde40889a076cb2c7f1612144998b1d388200bc7e295/analysis/">search result</a> confirm this is indeed the correctly extracted MiniDuke file.
