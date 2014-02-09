---
layout: post
title: Things I've learned using skulpt for in-browser Python code
categories: []
tags: []
status: publish
type: post
published: true
---
This post summarizes some core concepts in getting <a href="http://www.skulpt.org/">skulpt</a> (Python in the browser) to work.

My <a href="http://icebuddha.com/">IceBuddha</a> project uses <a href="http://www.skulpt.org/">skulpt</a> for in-browser Python code.  I wanted the ability to allow users to write their own parsing scripts that they could test immediately.  The solution I initially chose was to have them write JavaScript with the help of some features of a domain specific language that was parsed with <a href="http://pegjs.majda.cz/">PEG.js</a>.  Few security professionals code in JavaScript regularly though, and they were my target audience so I wanted to use Python.

Options:
<ol>
<li> Upload the Python code and binary to be parsed to my server, and have the Python code execute there with the binary as input and produce a result to send back to the user's browser.
<ul><li><b>Positives</b><ul>
<li>Easiest solution to implement
</ul></ul>
<ul><li><b>Negatives</b><ul>
<li>Security concerns of user provided data being executed on my server.
<li>Potentially large amounts of data being uploaded to my server (the binary files especially could be large).
</ul></ul>
<li> Leave the binary in the user's browser, and upload the Python code to my server, compile it to JavaScript using <a href="http://pyjs.org/">pyjs</a> (formerly called Pyjamas) or some other option, and send the JavaScript back to the user's browser.
<ul><li><b>Positives</b><ul>
<li>Possibly a more featureful implementation of Python than what Skulpt supports
</ul></ul>
<ul><li><b>Negatives</b><ul>
<li>I wanted as few dependencies on my server as possible, and for privacy concerns of users I didn't want the user to upload anything to my server.
</ul></ul>
<li> Leave the binary and the Python code on the users browser, and compile the Python code to JavaScript locally using Skulpt.
<ul><li><b>Positives</b><ul>
<li>Completely in-browser solution.
</ul></ul>
<ul><li><b>Negatives</b><ul>
<li>Using Skulpt requires the user to download between 242K-440K of JavaScript.
<li>Imported modules have to be compiled to JavaScript in advance, or you could do a hack of concat'ing files together.
<li>Some Python functionality is not supported.
<li>Debugging Python code that has been compiled to JavaScript can be tricky, especially when the problem may simply be that the feature you want to use has not been added to Skulpt yet.
</ul></ul>
</ol>

For my needs, I decided to go with Skulpt.

<h3>Hello World with Skulpt</h3>
There is a great example on using skulpt on the homepage of <a href="http://www.skulpt.org">skulpt.org</a>, but it does graphics and thus a little extra than is really needed, so I'm going to simplify this further.
<ol>
<li>Download <a href="https://github.com/bnmnetp/skulpt/raw/master/dist/skulpt.js">skulpt.js</a> (242KB)
<li>Add the following to a new file called test.html
{% highlight html %}
<html> 
<head>
<script src="skulpt.js" type="text/javascript"></script> 
</head> 

<body> 
<textarea id="pythonCode">
for i in range(10):
    print i
</textarea><br /> 
<pre id="output"></pre> 

<script type="text/javascript"> 
function outf(text) { 
    var mypre = document.getElementById("output"); 
    mypre.innerHTML = mypre.innerHTML + text; 
} 

var code = document.getElementById("pythonCode").value; 
Sk.configure({output:outf}); 
eval(Sk.importMainWithBody("<stdin>",false,code)); 
</script> 

</body> 
</html> 
{% endhighlight %}

<li>Open test.html in your browser, and it should display the Python code and then the numbers 0 through 9. Hurray!
</ol>
The JavaScript at the bottom finds the code we want to run (in the textarea), tells skulpt to use outf as our stdout when Python calls print, and then runs the code.

<h3>Loading Python code from files on the server</h3>
For me, the first thing that bothered me when working with skulpt, is I didn't want to have my Python code in my html or JavaScript files.  I wanted just Python in .py files.  This way all my syntax high-lighting and <a href="http://www.python.org/dev/peps/pep-0008/">PEP8</a> checking would work.  My solution is in my JavaScript code, I use jquery to grab my file:
{% highlight javascript %}
cacheBreaker = "?"+new Date().getTime();
$.get("./test.py"+cacheBreaker, function(response) {
    runSkulpt(response);
});
{% endhighlight %}

<h3>Passing data to the Python code</h3>
To pass in data, I make my Python code a class, and then do things slightly differently.  Here is an example:

Javascript code
{% highlight javascript %}
var module = Sk.importMainWithBody("<stdin>", false, code);
var obj = module.tp$getattr('MyClass');
var runMethod = obj.tp$getattr('run');

var arrayForSkulpt = new Array();
for (var i=0; i<5; i++) {
  arrayForSkulpt[i] = i;
}

// Run parse script
var ret = Sk.misceval.callsim(runMethod, Sk.builtin.list(arrayForSkulpt));
{% endhighlight %}

Python code
{% highlight python %}
class MyClass:
    def run(self, data):
        for i in data:
            print i
{% endhighlight %}

<h3>Getting data back from the Python code</h3>
I had trouble returning anything other than arrays, strings, numbers, and arrays of arrays back from the Python.  So in my Python code I just return some arrays of data from the run method, and then in my JavaScript code, I spent a lot of time with the JavaScript debugger to figure out how to get the data back I wanted.  The trick is that the data you want will be in <tt>ret.v</tt> in that JavaScript code from the last example.  So if you change the Python code to:
{% highlight python %}
class MyClass:
    def run(self, data):
        sum = 0
        for i in data:
            sum += i
        return sum
{% endhighlight %}

Then in your JavaScript you'll have:
{% highlight javascript %}
// Run parse script
var ret = Sk.misceval.callsim(runMethod, Sk.builtin.list(arrayForSkulpt));
var sum = ret.v;
{% endhighlight %}


<h3>Adding modules to skulpt</h3>
Depending on your needs, you may want to just concatenate files together and run that blob through Skulpt.  If you want to do more legit modules, then you'll need to compile those in advance, and this will require you to use an additional file (which you will build) called <tt>builtin.js</tt> that will be about 200K and will need to be sent to users of your site.

<ol>
<li>Download the Skulpt code: <tt>git clone https://github.com/bnmnetp/skulpt.git</tt>
<li>Copy your module directory to <tt>./skulpt/src/lib/</tt>.  For example:
{% highlight bash %}
cd skulpt
mkdir ./src/lib/MyModule
cat >./src/lib/MyModule/__init__.py <<EOL
class MessagePrinter:
    def printMessage(self):
        print "Hello World"
EOL
{% endhighlight %}
<li>Compile skulpt and copy the new files to your server:
{% highlight bash %}
./m dist
cp dist/* yourWebServer/.
{% endhighlight %}

<li>At the top of your HTML file, you'll need to add a reference to the file <tt>builtin.js</tt> after <tt>skulpt.js</tt>, so it looks like:
{% highlight html %}
<html> 
<head>
<script src="skulpt.js" type="text/javascript"></script> 
<script src="builtin.js" type="text/javascript"></script> 
</head>
{% endhighlight %}

<li>In your JavaScript code, you'll need to add in the function <tt>builtinRead</tt> and add it to your <tt>Sk.configure</tt> call:
{% highlight javascript %}
function builtinRead(x) {
    if (Sk.builtinFiles === undefined || Sk.builtinFiles["files"][x] === undefined)
            throw "File not found: '" + x + "'";
    return Sk.builtinFiles["files"][x];
}

Sk.configure({output:outf, read:builtinRead}); 
{% endhighlight %}

<li>Change your Python to use the new module:
{% highlight python %}
import MyModule

class MyClass:
    def run(self, data):
        mp = MyModule.MessagePrinter()
        mp.printMessage()
{% endhighlight %}
</ol>
