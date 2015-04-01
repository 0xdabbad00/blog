---
layout: post
title: ! 'File scanner web app (Part 2 of 5): Upload files'
categories: []
tags: []
status: publish
type: post
published: true
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

{% highlight html %}
<html>
<head>
  <script src="/dropzone.js"></script>

  <style>
  .dropzone {
    border-style:dotted;
    border-width:2px;
    min-height: 100px;
    height:100px;
  }
  </style>
</head>

<body>
  <h1>File Scanner</h1>

  <form action="/file-upload"
        class="dropzone"
        id="mydropzone"></form>
</body>

<script>
Dropzone.options.mydropzone = {
  previewTemplate : '<div class="preview file-preview">\
  <div class="dz-details">\
    <b class="dz-filename"><span data-dz-name></span></b>\
    <b class="dz-size" data-dz-size></b>\
  </div>\
  <div class="dz-progress"><span class="dz-upload" data-dz-uploadprogress></span></div>\
  <div class="dz-error-message"><span data-dz-errormessage></span></div>',
  init: function() {
    this.on("complete", function(file) { console.log("Upload complete"); });
  }
};
</script>
{% endhighlight %}

Edit <tt>webserver.py</tt> to contain the following and restart the process.

<!-- highlight="10,11,12,13,14,15,18" -->

{% highlight python linenos=table %}
#!/usr/bin/python

import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render("static/index.htm")


class UploadHandler(tornado.web.RequestHandler):             # +
    def post(self):                                          # +
        file_contents = self.request.files['file'][0].body   # +
        with open("uploads/file", "wb") as f:                # +
            f.write(file_contents)                           # +
        self.finish()                                        # +


application = tornado.web.Application([
    (r"/file-upload", UploadHandler),                        # +
    (r"/", MainHandler),
    (r'/(.*)', tornado.web.StaticFileHandler, {'path': 'static'}),
])

if __name__ == "__main__":
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
{% endhighlight %}

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

<!-- highlight="14,16,18,19,20,21,23,27,28,29,30,31") -->

{% highlight python linenos=table %}
#!/usr/bin/python

import tornado.ioloop
import tornado.web

import MySQLdb

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render("static/index.htm")


class UploadHandler(tornado.web.RequestHandler):
    def post(self):
        file_name = self.request.files['file'][0].filename
        file_contents = self.request.files['file'][0].body
        file_size = len(file_contents)

        stmt = "INSERT INTO files (filename, size) VALUES (%(filename)s, %(filesize)s)"
        cur.execute(stmt, {'filename': file_name, 'filesize': file_size})
        file_id = cur.lastrowid
        db.commit()

        with open("uploads/%s" % file_id, "wb") as f:
            f.write(file_contents)
        self.finish()


db = MySQLdb.connect(host="localhost",
                     user="root",
                      passwd="mypassword",
                      db="scanner")
cur = db.cursor()

application = tornado.web.Application([
    (r"/file-upload", UploadHandler),
    (r"/", MainHandler),
    (r'/(.*)', tornado.web.StaticFileHandler, {'path': 'static'}),
])

if __name__ == "__main__":
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
{% endhighlight %}

We'll fill in the MD5 hash later.  For now, try uploading some files, and ensure they should up with ID's for names in <tt>uploads</tt>, and you can check the mysql using <tt>select * from files</tt>.

<h3>Conclusion</h3>
You can now upload files to your webserver and have some info about them stored in the database.  In order to have something interesting happen with the file, you'll need to complete the rest of the series.

Next <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-3-of-5-yara-signatures/">Part 3</a>
