---
layout: post
title: ! 'File scanner web app (Part 2 of 5): Upload files'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1378167016'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  _oembed_d84d71c47ba0da28650347484a69ebb6: ! '{{unknown}}'
---
<a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-1-of-5-stand-up-and-webserver/">Part 1</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-2-of-5-upload-files/">Part 2</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-3-of-5-yara-signatures/">Part 3</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-4-of-5-scanning-files-from-the-web-app/">Part 4</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-5-of-5-finishing-touches/">Part 5</a>

So far in this dev training, we stood up an Ubuntu VM and made a basic web service at <tt>/var/apps/scanner/webserver.py</tt>.  Now we'll change it from a simple static web server to something that can upload files.

<h3>Upload files using DropzoneJS</h3>
We'll use <a href="http://www.dropzonejs.com/">Dropzone.js</a> for uploading files.  Get a copy by running:
<pre>
wget -O /var/apps/scanner/static/dropzone.js \
https://raw.github.com/enyo/dropzone/master/downloads/dropzone.js
</pre>

Now, let's edit our ugly <tt>index.htm</tt> to do something.

[sourcecode lang="html" gutter="false"]
&lt;html&gt;
&lt;head&gt;
	&lt;script src=&quot;/dropzone.js&quot;&gt;&lt;/script&gt;

	&lt;style&gt;
	.dropzone {
		border-style:dotted; 
		border-width:2px;
		min-height: 100px;
		height:100px;
	}
	&lt;/style&gt;
&lt;/head&gt;

&lt;body&gt;
	&lt;h1&gt;File Scanner&lt;/h1&gt;

	&lt;form action=&quot;/file-upload&quot;
	      class=&quot;dropzone&quot;
	      id=&quot;mydropzone&quot;&gt;&lt;/form&gt;
&lt;/body&gt;

&lt;script&gt;
Dropzone.options.mydropzone = {
  previewTemplate : '&lt;div class=&quot;preview file-preview&quot;&gt;\
  &lt;div class=&quot;dz-details&quot;&gt;\
    &lt;b class=&quot;dz-filename&quot;&gt;&lt;span data-dz-name&gt;&lt;/span&gt;&lt;/b&gt;\
    &lt;b class=&quot;dz-size&quot; data-dz-size&gt;&lt;/b&gt;\
  &lt;/div&gt;\
  &lt;div class=&quot;dz-progress&quot;&gt;&lt;span class=&quot;dz-upload&quot; data-dz-uploadprogress&gt;&lt;/span&gt;&lt;/div&gt;\
  &lt;div class=&quot;dz-error-message&quot;&gt;&lt;span data-dz-errormessage&gt;&lt;/span&gt;&lt;/div&gt;',
  init: function() {
    this.on(&quot;complete&quot;, function(file) { console.log(&quot;Upload complete&quot;); });
  }
};
&lt;/script&gt;
[/sourcecode] 

Edit <tt>webserver.py</tt> to contain the following and restart the process.
[sourcecode lang="python" gutter="false" highlight="10,11,12,13,14,15,18"]
#!/usr/bin/python
 
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render(&quot;static/index.htm&quot;)


class UploadHandler(tornado.web.RequestHandler):
    def post(self):
        file_contents = self.request.files['file'][0].body
        with open(&quot;uploads/file&quot;, &quot;wb&quot;) as f:
            f.write(file_contents)
        self.finish()

 
application = tornado.web.Application([
    (r&quot;/file-upload&quot;, UploadHandler),
    (r&quot;/&quot;, MainHandler),
    (r'/(.*)', tornado.web.StaticFileHandler, {'path': 'static'}),
])
 
if __name__ == &quot;__main__&quot;:
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
[/sourcecode]

Now you can either drag and drop files to your webpage (inside the dotted box) or click inside the dotted box and it will pop-up a file explorer view so you can select a file to upload.  The file will be uploaded as <tt>/var/apps/scanner/uploads/file</tt>.  So now we'll need to do something to give unique filenames.

<h3>Setting up MySQL</h3>
Install MySQL and Python's mysqldb library by running:
<pre>
sudo apt-get install mysql-server python-mysqldb
</pre>

For this guide, I'll use the password <tt>mypassword</tt>.  From the command line run:
<pre>
mysql -u root -p
</pre>
Provide the password when prompted.  Now we'll create a database and our initial table.
<pre>
create database scanner;
use scanner;
create table files (
  id INT NOT NULL AUTO_INCREMENT,
  submission_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  filename VARCHAR(30) NOT NULL DEFAULT "", 
  size INT NOT NULL DEFAULT 0, 
  md5 CHAR(32) NOT NULL DEFAULT "", 
  PRIMARY KEY(id)
);
</pre>

Initially, all we'll do in our database is store the filename given to us, and we'll figure out the size and md5 for it.  On disk, we'll save the files as <tt>uploads/1</tt>, <tt>uploads/2</tt>, etc.  Doing this ensures we don't run into naming collisions.

<h3>Upload files with unique names</h3>
Now we need to modify our <tt>webserver.py</tt> to use our database.

[sourcecode lang="python" gutter="false" highlight="14,16,18,19,20,21,23,27,28,29,30,31"]
#!/usr/bin/python
 
import tornado.ioloop
import tornado.web

import MySQLdb

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render(&quot;static/index.htm&quot;)


class UploadHandler(tornado.web.RequestHandler):
    def post(self):
        file_name = self.request.files['file'][0].filename
        file_contents = self.request.files['file'][0].body
        file_size = len(file_contents)

        stmt = &quot;INSERT INTO files (filename, size) VALUES (%(filename)s, %(filesize)s)&quot;
        cur.execute(stmt, {'filename': file_name, 'filesize': file_size})
        file_id = cur.lastrowid
        db.commit()

        with open(&quot;uploads/%s&quot; % file_id, &quot;wb&quot;) as f:
            f.write(file_contents)
        self.finish()


db = MySQLdb.connect(host=&quot;localhost&quot;,
                     user=&quot;root&quot;,
                      passwd=&quot;mypassword&quot;,
                      db=&quot;scanner&quot;)
cur = db.cursor()
 
application = tornado.web.Application([
    (r&quot;/file-upload&quot;, UploadHandler),
    (r&quot;/&quot;, MainHandler),
    (r'/(.*)', tornado.web.StaticFileHandler, {'path': 'static'}),
])
 
if __name__ == &quot;__main__&quot;:
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
[/sourcecode]

We'll fill in the MD5 hash later.  For now, try uploading some files, and ensure they should up with ID's for names in <tt>uploads</tt>, and you can check the mysql using <tt>select * from files</tt>.

<h3>Conclusion</h3>
You can now upload files to your webserver and have some info about them stored in the database.  In order to have something interesting happen with the file, you'll need to complete the rest of the series.

Next <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-3-of-5-yara-signatures/">Part 3</a>
