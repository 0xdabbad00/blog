---
layout: post
title: ! 'File scanner web app (Part 1 of 5): Stand-up and webserver'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1378167213'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
If you're like me, your daily in-take of infosec news high-lights new vulns, things broken, and maybe some reverse engineering of some malware, but my day-to-day life is as a software developer in the infosec world.  I build software, and I'm going to show you how to quickly stand up something useful using many of the technologies I use in my day-to-day life.

In this series I'm going to show how to quickly stand-up a web app that you can drag and drop files to that will get scanned using some <a href="https://code.google.com/p/yara-project/">YARA</a> signatures.  The value of this series is not so much in the end product as it is in learning how all these tools fit together and how you can build things quickly, with some polish along the way so your projects aren't just hacks that fall apart on the first power failure or when someone else tries to use it.  A lot of development today, especially for web-apps, is just gluing different components together:  Half the battle is knowing what to use to accomplish your goals, and the other half is just applying the glue.  This will teach some of both skills.

To give you a better idea of what you'll be making, here is a screenshot of the final product.
<a href="http://0xdabbad00.com/wp-content/uploads/2013/09/Screen-Shot-2013-09-02-at-3.30.33-PM.png"><img src="http://0xdabbad00.com/wp-content/uploads/2013/09/Screen-Shot-2013-09-02-at-3.30.33-PM-300x152.png" alt="" title="Finished app" width="300" height="152" class="aligncenter size-medium wp-image-1190" /></a>

You can see the final code by looking <a href="https://github.com/0xdabbad00/filescannner_webapp">here</a>, but there is a lot of setup needed along the way, maybe I'll write an installer if this gets some attention.

People unfamiliar with software development think it's about knowing certain languages. That's one small part of the discipline.  It's more about knowing certain library APIs, OS functionality, how to use tools, how to debug, and lots of other skills.

Skills you'll learn:
<ul>
<li><a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-1-of-5-stand-up-and-webserver/">Part 1</a>: Stand-up and webserver
<ul>
<li>Set-up and interact with a Ubuntu VM on VirtualBox through more than the console display, using <a href="http://fuse.sourceforge.net/sshfs.html">sshfs</a>.
<li>Python's <a href="http://www.tornadoweb.org/en/stable/">Tornado</a> library for creating a web server
</ul>
<li><a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-2-of-5-upload-files/">Part 2</a>: Upload files
<ul>
<li><a href="http://www.dropzonejs.com/">DropzoneJS</a> for dragging and dropping files to a web app 
<li><a href="http://www.mysql.com/">MySQL</a> for a back-end database
</ul>
<li><a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-3-of-5-yara-signatures/">Part 3</a>: YARA signatures
<ul>
<li>Creating <a href="https://code.google.com/p/yara-project/">YARA</a> signatures
<li>Integrating YARA with Python
</ul>
<li><a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-4-of-5-scanning-files-from-the-web-app/">Part 4</a>: Scanning files from the web app
<ul>
<li><a href="http://www.rabbitmq.com/">RabbitMQ</a> for work queuing.
<li>Returning data as JSON data from a web app so AJAX calls can be made on it easily.
</ul>
<li><a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-5-of-5-finishing-touches/">Part 5</a>: Finish it
<ul>
<li>Twitter's <a href="http://getbootstrap.com/">Bootstrap</a> library for making pretty web pages
<li>JQuery <a href="https://datatables.net/">Datatables</a> for a pretty table display, with connections for JSON data
<li>Using <a href="http://supervisord.org/">Supervisord</a> for services monitoring and control.
</ul>
</ul>

<h3>Standing up the environment</h3>
I'll be using VirtualBox and a VM for Ubuntu Server 12.04 x64.
<ol>
<li><b>Install <a href="https://www.virtualbox.org/wiki/Downloads">VirtualBox</a>.</b>
<li><b>Create an Ubuntu VM</b>.  I downloaded the <a href="http://www.ubuntu.com/download/server/thank-you?distro=server&bits=64&release=lts">Ubuntu Server 12.04, 64-bit</a> iso.  Create the VM, with all defaults, and make it an SSH server.
<li><b>Snapshot</b> You should always have a clean Ubuntu install ready to play with, so take a snapshot.
<li><b>Add a Host-Only network adapter</b>  VirtualBox can be a pain to connect to.  The default network setting is NAT, which allows the guest to access the Internet, but you can't connect to the guest from the host.  To connect to the guest from the host, you can do one of the following:
<ul>
<li>Set up port-forwarding: Painful because you have to remember the port mapping (8022 on your localhost is 22 on the guest, or something like that).
<li>Change the network adapter to Host-only from NAT, so you no longer have access to the Internet from the guest, which means you can't download anything without first downloading it to your host then scp'ing or otherwise copying it over.
<li>Add a Host-only network adapter.  No negatives.
</ul>
Let's add the host-only network adapter.  If the following instructions get confusing, look at <a href="http://christophermaier.name/blog/2010/09/01/host-only-networking-with-virtualbox">Host-Only Networking With VirtualBox</a>
<ol>
<li>In VirtualBox's main menu, go to your preferences.  Click Network, and add a Host-only network (click the plus sign and something like "vboxnet0" should appear if you didn't have it already).  See <a href="http://superuser.com/questions/429405/how-can-i-get-virtualbox-to-run-with-a-hosts-only-adapter">this</a> if you have problems.
<li>In your VM, go to it's settings and add a second adapter for Host-Only networking.  You'll need to shutdown the VM to do this.
<li>Start the VM up, and if you run <tt>ifconfig</tt> you'll notice you don't have a second adapter.  Run the following to turn it on:
<pre>
sudo ifconfig eth1 192.168.56.101 netmask 255.255.255.0 up
</pre>
That is a temporary fix that will not survive reboot.  You can now ssh in though.  To make this change permanent edit the file <tt>/etc/network/interfaces</tt> to add the following:
<pre>
# The host-only network interface
auto eth1
iface eth1 inet static
address 192.168.56.101
netmask 255.255.255.0
network 192.168.56.0
broadcast 192.168.56.255
</pre>
</ol>
You can now ssh in.  Do that, because working through VitualBox's console display is annoying.  You can also edit your <tt>/etc/hosts</tt> file now if you want so you don't have to remember the IP address.
<li><b>Update it.</b> You're in infosec, so make sure your stuff is patched.  Everything should be up-to-date if you just downloaded the latest, but run the following anyway:
<pre>
sudo apt-get update
sudo apt-get dist-upgrade
</pre>
</ol>

<h3>SSHFS: Get yourself a way to edit files</h3>
As legit as a console and vim are, it's 2013, and using a GUI text editor like <a href="http://www.sublimetext.com/">Sublime Text</a> can make life nicer.  That means being able to access files as if they were local.  You could setup an SMB mount, but let's use sshfs.  If you're on Linux, you can follow <a href="http://www.howtogeek.com/howto/ubuntu/how-to-mount-a-remote-folder-using-ssh-on-ubuntu/">this</a>, or on Windows try <a href="http://www.linux-wizard.net/2011/07/11/using-dokan-under-windows-to-mount-your-home-with-ssh/">this</a>, but I'm on Mac OSX, so I installed OSXFUSE and then SSHFS from <a href="http://osxfuse.github.io/">here</a>.  
On the guest I ran:
<pre>
mkdir /var/apps
chmod 777 /var/apps
</pre>
From my host, I then ran:
<pre>
mkdir /Volumes/apps
sshfs user@192.168.56.101:/ /Volumes/apps
</pre>
You'll be able to access your guest's files now, but in order to get the /Volumes to show in your host's Finder, run the following:
<pre>
sudo SetFile -a v /Volumes
</pre>

<h3>Tornado: Make a web server</h3>
All that work and we still have nothing to show for it.  Let's make a web server.  We'll use python's Tornado library.  On the guest run:
<pre>
sudo apt-get install python-setuptools
sudo easy_install tornado
</pre>

Create the directory <tt>/var/apps/scanner</tt> to work in.  Create a file there named <tt>webserver.py</tt> with the following contents:
[sourcecode lang="python" gutter="false"]
#!/usr/bin/python

import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write(&quot;Hello world&quot;)

application = tornado.web.Application([
    (r&quot;/&quot;, MainHandler),
])

if __name__ == &quot;__main__&quot;:
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
[/sourcecode]

Run it with:
<pre>
python webserver.py
</pre>

From your host, you should now be able to visit <a href="http://192.168.56.101:8000/">http://192.168.56.101:8000/</a>.  You'll see a simple "Hello world" displayed.

Tornado allows you to quickly create web services.  You could do something like this with <a href="http://httpd.apache.org/">Apache</a> and <a href="http://modpython.org/">mod_python</a>, or with Python's <a href="http://twistedmatrix.com/trac/">twisted</a> library, or <a href="http://www.gevent.org/">gevent</a>.  In software development there are often many options to choose from at each step.  Tornado I've found is quick and easy, although with that you lose some things like the default logging that Apache has.  Also, all python apps are single-threaded, so you have to be more mindful not to do any actions that would cause blocking, like slow database queries.

Reading our code from the bottom up, it is simply saying "Create a web server on port 8000.  Anything that tries to get the page '/' should see whatever the MainHandler returns.  The MainHandler simply writes 'Hello world'".

<h3>Turn it into a static file server</h3>
Right now, your webserver can't display any files.  It can only respond with "Hello world" and only if the client  tries to access "/".  Otherwise it gives a 404.  Let's turn our server into a static file server by killing the "python webserver.py" process, replacing the <tt>webserver.py</tt> file with the following and starting the process back up.
[sourcecode lang="python" gutter="false" highlight="8,12"]
#!/usr/bin/python
 
import tornado.ioloop
import tornado.web
 
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render(&quot;static/index.htm&quot;)
 

application = tornado.web.Application([
    (r&quot;/&quot;, MainHandler),
    (r'/(.*)', tornado.web.StaticFileHandler, {'path': 'static'}),
])
 
if __name__ == &quot;__main__&quot;:
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
[/sourcecode]

Now create a directory <tt>/var/apps/scanner/static</tt> and create a <tt>index.htm</tt> file there.  For example:
[sourcecode gutter="false"]
echo &quot;&lt;html&gt;&lt;body bgcolor=yellow&gt;This is static&quot; &gt; /var/apps/scanner/static/index.htm
[/sourcecode]

After starting the service back up, visit <a href="http://192.168.56.101:8000/">http://192.168.56.101:8000/</a> and you should see an ugly site with your text.

Tornado caches your files, so any time you edit that static <tt>index.htm</tt> file or any other file you plan on serving, you'll need to restart <tt>webserver.py</tt>.

<h3>Handlers</h3>
To give you a feel for how handlers in Tornado work, let's add two additional ones to our code.
[sourcecode lang="python" gutter="false" highlight="5,11,12,13,15,16,17,20,21"]
#!/usr/bin/python
 
import tornado.ioloop
import tornado.web
from datetime import datetime

 
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render(&quot;static/index.htm&quot;)

class TimeHandler(tornado.web.RequestHandler):
    def get(self):
        self.write(&quot;%s&quot; % datetime.now())
 
class AdditionHandler(tornado.web.RequestHandler):
    def get(self, num1, num2):
        self.write(&quot;%d&quot; % (int(num1)+int(num2)))

 
application = tornado.web.Application([
    (r&quot;/time&quot;, TimeHandler),
    (r&quot;/add/([0-9]+)\+([0-9]+)&quot;, AdditionHandler),
    (r&quot;/&quot;, MainHandler),
    (r'/(.*)', tornado.web.StaticFileHandler, {'path': 'static'}),
])
 
if __name__ == &quot;__main__&quot;:
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
[/sourcecode]

The first new handler (<tt>TimeHandler</tt>) at <a href="http://192.168.56.101:8000/time">http://192.168.56.101:8000/time"</a> shows you how easy it is to get your own python code running whenever the client visits one of our registered handlers.

The other handler (<tt>AdditionHandler</tt>) at <a href="http://192.168.56.101:8000/add/3+15">http://192.168.56.101:8000/add/3+15</a> so you can see how to include parameters in your URL's and how those are interpreted by Tornado.  The handlers are registered using regex's, and the "variables" of those regex's get included as additional parameters to the <tt>get</tt> method.

Play around and make some different handlers!

Next up, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-2-of-5-upload-files/">Part 2</a>
