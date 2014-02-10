---
layout: post
title: IceBuddha scrolling (javascript infinite scrolling in a finite area)
categories: []
tags: []
status: publish
type: post
published: true
---
One of the tricky parts in building IceBuddha is that it needs to be able to display the hexdump for an entire binary.  My goal has been to be able to do this for any binary under 10MB.  IceBuddha reads the entire binary into memory into an array called data.  To display a hexdump, my first solution was to just convert this to to html with something like:
{% highlight html %}
<pre>
00000000h 4D 5A 90 00 03 00 00 00 04  00 00 00 FF FF 00 00 MZ..............
00000010h B8 00 00 00 00 00 00 00 40  00 00 00 00 00 00 00 ........@.......
</pre>
{% endhighlight %}

I wanted to be able to high-lite bytes though, which gives two options:
<ul>
<li><b>Solution 1: Sub-string and refresh</b><br>I could recreate the html with my desired data inside a DOM element with it's own high-liting.
{% highlight html %}
<pre>
00000000h <i id="highlite">4D 5A</i> 90 00 03 00 00 00 04  00 00 00 FF FF 00 00 MZ..............
</pre>
{% endhighlight %}

<li><b>Solution 1: Lots of DOM elements</b><br>
I could make each hex value it's own DOM element.
{% highlight html %}
<pre>
00000000h 
<i id="h0">4D</i> 
<i id="h1">5A</i> 
<i id="h2">90</i> 
<i id="h3">00</i> 
<i id="h4">03</i> 
<i id="h5">00</i> 
<i id="h6">00</i> 
<i id="h7">00</i> 
<i id="h8">04</i> 
<i id="h9">00</i> 
<i id="h10">00</i> 
<i id="h11">00</i> 
<i id="h12">FF</i> 
<i id="h13">FF</i> 
<i id="h14">00</i> 
<i id="h15">00</i> 
MZ..............
</pre>
{% endhighlight %}
If I want to flip on high-liting somewhere, all I need to do is:
{% highlight javascript %}
$("#h"+i).css("background", color);
{% endhighlight %}
</ul>

I decided to go with the second option, because it seemed easier to work with.  

The problem is for a 1MB file, I end up with over 10 million DOM elements, plus I actually have DOM elements for all the ASCII, so 20 million.  This many DOM elements freezes your browser.  Looking around online, 10K elements is a good rule of thumb absolute max, and most places recommending you stay under 1K.

I decided to ignore this problem for a long time and just get the rest of the app functional, and when any file is loaded it truncates the file to 2K.  Obviously I would need to fix this eventually, and so I have.  My solution was to implement what will be very familiar to game programmers: <b>Only draw what is going to be displayed</b>.  I keep track of what the user is trying to view, and only create html for that data.

For those that want to follow along, I made a tag of my code on github called "scrollable" which allows you to download <a href="https://github.com/0xdabbad00/icebuddha/archive/scrollable.zip">icebuddha-scrollable.zip</a>.  This is commit <a href="https://github.com/0xdabbad00/icebuddha/tree/6856d57c616dfae11d889a60b91193c55419161d">6856d57c616dfae11d889a60b91193c55419161d</a>.

I need to refactor badly, but here is what is going on.  All the hex data is displayed in a div called `#byte_content`, which has the css `max-height: 200px; overflow-y: scroll;` applied to it, which means this is the "view window" and will have a scroll bar.

In `drop.js` there is a function `getByteContentHTML(...)` that creates a giant table that is mostly going to be empty:

{% highlight javascript %}
tableHeight = data.length/BYTES_PER_LINE * FONT_HEIGHT;
preHeight = start/BYTES_PER_LINE * FONT_HEIGHT;

tableHeightStyle = "style=\"min-height:"+tableHeight+"px; height:"+tableHeight+"px; border-spacing: 0px;\"";
preHeightStyle = "style=\"min-height:"+preHeight+"px; height:"+preHeight+"px;\"";

output.push("<table border=0 cellpadding=0 cellspacing=0 "+tableHeightStyle+" id=\"byteScrollableArea\">");
output.push("<tr "+preHeightStyle+"><td "+preHeightStyle+" id=\"byteFillerAbove\"><td><td></tr>");
output.push("<tr>....");
{% endhighlight %}

So I make a giant table (`#byteScrollableArea`), and the first row (`#byteFillerAbove`) is going to be empty and includes whatever is above the drawn data (`#hexCell` and some other stuff), then the data gets written which will extend above and below the currently visible area, and then the table just extends down to the bottom of whatever is left of the file.  I tried to diagram this.
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

Whenever I draw the data to the screen, in `displayHexDump`, I do the following:
{% highlight javascript %}
$('#byte_content').unbind('scroll', outOfRangeScrollHandler);

$('#byte_content').scrollTo($("#h"+position), 1, {onAfter:function(){
  SetupScrollDownWaypoint();
  SetupScrollUpWaypoint();

  // If the user grabs the scroll bar, make sure to refresh the screen
  $('#byte_content').bind('scroll', outOfRangeScrollHandler);
}});
{% endhighlight %}

The functions `SetupScrollDownWaypoint();` and `SetupScrollUpWaypoint();` don't actually exist, but that is what the code is doing.  I used these waypoints a lot originally, but then I realized that if the user just grabs the scroll bar and yanks it to the bottom of the file, you can jump past the way point and no data get's displayed, so I had to create the function `outOfRangeScrollHandler` to see if the user was displaying our data or had scrolled outside of it (the waypoint code may no longer be necessary due to this function).

{% highlight javascript %}
var outOfRangeScrollHandler = function() {
	scrollPos = $('#byte_content').scrollTop();
	topOfContent = $('#byteFillerAbove').height();
	contentHeight = FONT_HEIGHT * LINES_TO_DISPLAY;

	if ((scrollPos < topOfContent - (FONT_HEIGHT * 1)) || 
		(scrollPos > (topOfContent+contentHeight) + (FONT_HEIGHT * 1))) {
		scrollLocation = (scrollPos / FONT_HEIGHT) * BYTES_PER_LINE;

		scrollToByte(scrollLocation);
	}
};
{% endhighlight %}

I also do some checks if the mouse button is down or not to see if the user is still dragging the scroll bar around.

Once the code decides the user has scrolled outside the screen, it calls a common function I wrote called `scrollToByte(scrollLocation);`.  This checks if the scroll location is outside of the currently painted data, and if it is, then repaint the data (set up waypoints again, etc.) and go there, else, smoothly scroll to the location with jQuery.ScrollTo.

{% highlight javascript %}
function scrollToByte(start) {
	if (hexDumpStart > start || hexDumpEnd < start) {
		displayHexDump(start - start % BYTES_PER_LINE);
	} else {
		var location = $("#h"+start);
		$('#byte_content').scrollTo(location, 800);
	}
}
{% endhighlight %} 
