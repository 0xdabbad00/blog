---
layout: post
title: IceBuddha scrolling (javascript infinite scrolling in a finite area)
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1354900608'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
One of the tricky parts in building IceBuddha is that it needs to be able to display the hexdump for an entire binary.  My goal has been to be able to do this for any binary under 10MB.  IceBuddha reads the entire binary into memory into an array called data.  To display a hexdump, my first solution was to just convert this to to html with something like:
[sourcecode language="html" gutter="false"]
&lt;pre&gt;
00000000h 4D 5A 90 00 03 00 00 00 04  00 00 00 FF FF 00 00 MZ..............
00000010h B8 00 00 00 00 00 00 00 40  00 00 00 00 00 00 00 ........@.......
&lt;/pre&gt;
[/sourcecode]

I wanted to be able to high-lite bytes though, which gives two options:
<ul>
<li><b>Solution 1: Sub-string and refresh</b><br>I could recreate the html with my desired data inside a DOM element with it's own high-liting.
[sourcecode language="html" gutter="false"]
&lt;pre&gt;
00000000h &lt;i id=&quot;highlite&quot;&gt;4D 5A&lt;/i&gt; 90 00 03 00 00 00 04  00 00 00 FF FF 00 00 MZ..............
&lt;/pre&gt;
[/sourcecode]

<li><b>Solution 1: Lots of DOM elements</b><br>
I could make each hex value it's own DOM element.
[sourcecode language="html" gutter="false"]
&lt;pre&gt;
00000000h 
&lt;i id=&quot;h0&quot;&gt;4D&lt;/i&gt; 
&lt;i id=&quot;h1&quot;&gt;5A&lt;/i&gt; 
&lt;i id=&quot;h2&quot;&gt;90&lt;/i&gt; 
&lt;i id=&quot;h3&quot;&gt;00&lt;/i&gt; 
&lt;i id=&quot;h4&quot;&gt;03&lt;/i&gt; 
&lt;i id=&quot;h5&quot;&gt;00&lt;/i&gt; 
&lt;i id=&quot;h6&quot;&gt;00&lt;/i&gt; 
&lt;i id=&quot;h7&quot;&gt;00&lt;/i&gt; 
&lt;i id=&quot;h8&quot;&gt;04&lt;/i&gt; 
&lt;i id=&quot;h9&quot;&gt;00&lt;/i&gt; 
&lt;i id=&quot;h10&quot;&gt;00&lt;/i&gt; 
&lt;i id=&quot;h11&quot;&gt;00&lt;/i&gt; 
&lt;i id=&quot;h12&quot;&gt;FF&lt;/i&gt; 
&lt;i id=&quot;h13&quot;&gt;FF&lt;/i&gt; 
&lt;i id=&quot;h14&quot;&gt;00&lt;/i&gt; 
&lt;i id=&quot;h15&quot;&gt;00&lt;/i&gt; 
MZ..............
&lt;/pre&gt;
[/sourcecode]
If I want to flip on high-liting somewhere, all I need to do is:
[sourcecode language="javascript" gutter="false"]
$(&quot;#h&quot;+i).css(&quot;background&quot;, color);
[/sourcecode]
</ul>

I decided to go with the second option, because it seemed easier to work with.  

The problem is for a 1MB file, I end up with over 10 million DOM elements, plus I actually have DOM elements for all the ASCII, so 20 million.  This many DOM elements freezes your browser.  Looking around online, 10K elements is a good rule of thumb absolute max, and most places recommending you stay under 1K.

I decided to ignore this problem for a long time and just get the rest of the app functional, and when any file is loaded it truncates the file to 2K.  Obviously I would need to fix this eventually, and so I have.  My solution was to implement what will be very familiar to game programmers: <b>Only draw what is going to be displayed</b>.  I keep track of what the user is trying to view, and only create html for that data.

For those that want to follow along, I made a tag of my code on github called "scrollable" which allows you to download <a href="https://github.com/0xdabbad00/icebuddha/archive/scrollable.zip">icebuddha-scrollable.zip</a>.  This is commit <a href="https://github.com/0xdabbad00/icebuddha/tree/6856d57c616dfae11d889a60b91193c55419161d">6856d57c616dfae11d889a60b91193c55419161d</a>.

I need to refactor badly, but here is what is going on.  All the hex data is displayed in a div called <code>#byte_content</code>, which has the css <code>max-height: 200px; overflow-y: scroll;</code> applied to it, which means this is the "view window" and will have a scroll bar.

In <code>drop.js</code> there is a function <code>getByteContentHTML(...)</code> that creates a giant table that is mostly going to be empty:

[sourcecode language="javascript" gutter="false"]
tableHeight = data.length/BYTES_PER_LINE * FONT_HEIGHT;
preHeight = start/BYTES_PER_LINE * FONT_HEIGHT;

tableHeightStyle = &quot;style=\&quot;min-height:&quot;+tableHeight+&quot;px; height:&quot;+tableHeight+&quot;px; border-spacing: 0px;\&quot;&quot;;
preHeightStyle = &quot;style=\&quot;min-height:&quot;+preHeight+&quot;px; height:&quot;+preHeight+&quot;px;\&quot;&quot;;

output.push(&quot;&lt;table border=0 cellpadding=0 cellspacing=0 &quot;+tableHeightStyle+&quot; id=\&quot;byteScrollableArea\&quot;&gt;&quot;);
output.push(&quot;&lt;tr &quot;+preHeightStyle+&quot;&gt;&lt;td &quot;+preHeightStyle+&quot; id=\&quot;byteFillerAbove\&quot;&gt;&lt;td&gt;&lt;td&gt;&lt;/tr&gt;&quot;);
output.push(&quot;&lt;tr&gt;....&quot;);
[/sourcecode]

So I make a giant table (<code>#byteScrollableArea</code>), and the first row (<code>#byteFillerAbove</code>) is going to be empty and includes whatever is above the drawn data (<code>#hexCell</code> and some other stuff), then the data gets written which will extend above and below the currently visible area, and then the table just extends down to the bottom of whatever is left of the file.  I tried to diagram this.
<a href="http://0xdabbad00.com/wp-content/uploads/2012/11/scrollable_diagram.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/11/scrollable_diagram-300x215.png" alt="" title="Scroll area" width="300" height="215" class="alignnone size-medium wp-image-557" /></a>

The reason for all this, is I wanted the scrollbar to still give an indication of where the user is looking at in the file.  Are they looking at the start? The bottom?  If I just displayed the first 2K bytes, and then appended to these as they scrolled (like the infinite scrolling in twitter) you would have two problems:
<ol>
<li>The user would have no concept of how big the file was and where in the file they were looking.
<li>You would quickly approach the issue earlier of too many DOM elements.
</ol>

The second problem could be easily addressed by refreshing the data so 2K are only ever displayed, but then how do you scroll back up? and you still don't know where you are in the file.

<h3>Moving around</h3>
Now that I can display data from anywhere in the file, the next trick is how do you scroll around and get the data to refresh at the correct times, and maintain the location?  Two libraries come to the rescue:
<ul>
<li><a href="http://flesler.blogspot.no/2007/10/jqueryscrollto.html">jQuery.scrollTo</a> Scrolls to an element
<li><a href="http://imakewebthings.com/jquery-waypoints/">jQuery Waypoints</a> Used in infinite scrolling solutions to load data as we get to elements.
</ul>

Whenever I draw the data to the screen, in <code>displayHexDump</code>, I do the following:
[sourcecode language="javascript" gutter="false"]
$('#byte_content').unbind('scroll', outOfRangeScrollHandler);

$('#byte_content').scrollTo($(&quot;#h&quot;+position), 1, {onAfter:function(){
  SetupScrollDownWaypoint();
  SetupScrollUpWaypoint();

  // If the user grabs the scroll bar, make sure to refresh the screen
  $('#byte_content').bind('scroll', outOfRangeScrollHandler);
}});
[/sourcecode]

The functions <code>SetupScrollDownWaypoint();</code> and <code>SetupScrollUpWaypoint();</code> don't actually exist, but that is what the code is doing.  I used these waypoints a lot originally, but then I realized that if the user just grabs the scroll bar and yanks it to the bottom of the file, you can jump past the way point and no data get's displayed, so I had to create the function <code>outOfRangeScrollHandler</code> to see if the user was displaying our data or had scrolled outside of it (the waypoint code may no longer be necessary due to this function).

[sourcecode language="javascript" gutter="false"]
var outOfRangeScrollHandler = function() {
	scrollPos = $('#byte_content').scrollTop();
	topOfContent = $('#byteFillerAbove').height();
	contentHeight = FONT_HEIGHT * LINES_TO_DISPLAY;

	if ((scrollPos &lt; topOfContent - (FONT_HEIGHT * 1)) || 
		(scrollPos &gt; (topOfContent+contentHeight) + (FONT_HEIGHT * 1))) {
		scrollLocation = (scrollPos / FONT_HEIGHT) * BYTES_PER_LINE;

		scrollToByte(scrollLocation);
	}
};
[/sourcecode]

I also do some checks if the mouse button is down or not to see if the user is still dragging the scroll bar around.

Once the code decides the user has scrolled outside the screen, it calls a common function I wrote called <code>scrollToByte(scrollLocation);</code>.  This checks if the scroll location is outside of the currently painted data, and if it is, then repaint the data (set up waypoints again, etc.) and go there, else, smoothly scroll to the location with jQuery.ScrollTo.

[sourcecode language="javascript" gutter="false"]
function scrollToByte(start) {
	if (hexDumpStart &gt; start || hexDumpEnd &lt; start) {
		displayHexDump(start - start % BYTES_PER_LINE);
	} else {
		var location = $(&quot;#h&quot;+start);
		$('#byte_content').scrollTo(location, 800);
	}
}
[/sourcecode] 
