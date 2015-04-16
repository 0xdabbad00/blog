---
layout: post
title: ! 'File scanner web app (Part 4 of 5): Scanning files from the web app'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1378167105'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
<a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-1-of-5-stand-up-and-webserver/">Part 1</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-2-of-5-upload-files/">Part 2</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-3-of-5-yara-signatures/">Part 3</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-4-of-5-scanning-files-from-the-web-app/">Part 4</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-5-of-5-finishing-touches/">Part 5</a>

In the last part of this series we create a python script to scan files and update our DB.  In the previous parts we created a web app that you could upload files to.  Now it's time to combine those two components.  Our web app uses Tornado, which is single threaded, so if we just scanned files as they were uploaded from within that web app, we'd be blocking and our web app would appear dead until the file finished scanning.  For our current, small and specific set of rules, this isn't a problem, but for a large rule set, it'd be annoying.  So we'll use <a href="http://www.rabbitmq.com/">RabbitMQ</a> to do work queuing.




Our web app will receive files from users, and then queue a message in RabbitMQ to tell it that someone should scan that file.  We'll have a separate process running that will wait for messages from RabbitMQ telling it to scan a file.  It's a basic producer/consumer process.

<h3>Installing RabbitMQ</h3>
Install RabbitMQ using the following:
<pre>
sudo apt-get install rabbitmq-server python-pip git-core
sudo pip install pika==0.9.8
</pre>

<h3>The Producer</h3>
First we'll make our web app queue messages.  I'm also using our new <tt>utils.py</tt> library now.

<!-- highlight="7,8,29,30,31,32,33,34,35,36,37,39" -->

{% highlight python linenos=table %}
#!/usr/bin/python
# file: webserver.py

import tornado.ioloop
import tornado.web

# + Add these libraries
import utils
import pika

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

        # + Queue the file
        # Queue work message
        connection = pika.BlockingConnection(pika.ConnectionParameters(
                       'localhost'))
        channel = connection.channel()
        channel.queue_declare(queue='uploaded_files')
        channel.basic_publish(exchange='',
                      routing_key='uploaded_files',
                      body='%s' % file_id)
        connection.close()


cur, db = utils.connectToDB()

application = tornado.web.Application([
    (r"/file-upload", UploadHandler),
    (r"/", MainHandler),
    (r'/(.*)', tornado.web.StaticFileHandler, {'path': 'static'}),
])

if __name__ == "__main__":
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
{% endhighlight %}

Run the web server, upload a file, and check that our queue has one message in it by running the following:
<pre>
<b>sudo rabbitmqctl list\_queues</b>
Listing queues ...
uploaded\_files	1
...done.
</pre>

<h3>The Consumer</h3>
Now we'll fix up our file <tt>yarascanner.py</tt> to consume these work messages.

<!-- highlight="8,43,44,45,46,47,49,50,51,52,53,54,55,57,58" -->

{% highlight python linenos=table %}
#!/usr/bin/python
# file: yarascanner.py

import utils
import yara
import sys
from os import path
import pika

def scanFile(file_id):
    print "Scanning file %d" % file_id
    filename = '%s' % file_id
    matches = rules.match(path.join('uploads', filename))
    print matches

    for match in matches:
        stmt = "SELECT id FROM rules WHERE name = %(name)s AND enabled=1"
        cur.execute(stmt, {'name': match})
        rule_id = cur.fetchone()[0]

        stmt = "INSERT INTO matches (file_id, rule_id) VALUES (%(file_id)s, %(rule_id)s)"
        cur.execute(stmt, {'file_id': file_id, 'rule_id': rule_id})
        db.commit()

# Read rules from database
cur, db = utils.connectToDB()

stmt = "SELECT text FROM rules_text rt JOIN rules r ON rt.id = r.id WHERE r.enabled=1"
cur.execute(stmt)
storedRules = cur.fetchall()

# Join them
rulesText = ""
for rule in storedRules:
    rulesText += rule[0] + '\n'

# Load them into yara
rules = yara.compile(source=rulesText)


if len(sys.argv) > 1:
    scanFile(int(sys.argv[1]))

# RabbitMQ callback
def callback(ch, method, properties, body):
    print " [x] Received %r" % (body,)
    scanFile(int(body))
    ch.basic_ack(delivery_tag = method.delivery_tag)

# Consume messages from work queue
connection = pika.BlockingConnection(pika.ConnectionParameters(
               'localhost'))
channel = connection.channel()
channel.queue_declare(queue='uploaded_files')
channel.basic_consume(callback,
                      queue='uploaded_files')

print ' [*] Waiting for messages. To exit press CTRL+C'
channel.start_consuming()
{% endhighlight %}

<h3>MD5 hash the file</h3>
One of thing I want our <tt>yarascanner.py</tt> to do is MD5 hash the file.  Seems like a good idea to do it here instead of web server where we don't want to block.  You could make another service for this, with another queue, if you wanted, but I don't want to clutter up this lesson too much.

<!-- highlight="9,27,28,29,30,31,32,34,35,36,37" -->

{% highlight python linenos=table %}
#!/usr/bin/python
# file: yarascanner.py

import utils
import yara
import sys
from os import path
import pika
import md5

def scanFile(file_id):
    print ";Scanning file %d"; % file_id
    filename = '%s' % file_id
    filepath = path.join('uploads', filename)
    matches = rules.match(filepath)
    print matches

    for match in matches:
        stmt = ";SELECT id FROM rules WHERE name = %(name)s AND enabled=1";
        cur.execute(stmt, {'name': match})
        rule_id = cur.fetchone()[0]

        stmt = ";INSERT INTO matches (file_id, rule_id) VALUES (%(file_id)s, %(rule_id)s)";
        cur.execute(stmt, {'file_id': file_id, 'rule_id': rule_id})
        db.commit()

    # Get MD5 hash of file
    m = md5.new()
    with open(filepath, 'rb') as f:
        filedata = f.read()
    m.update(filedata)
    filehash = m.hexdigest()

    # Add the hash to the DB
    stmt = ";UPDATE files SET md5 = %(md5)s WHERE id = %(id)s";
    cur.execute(stmt, {'md5': filehash, 'id': file_id})
    db.commit()



# Read rules from database
cur, db = utils.connectToDB()

stmt = ";SELECT text FROM rules_text rt JOIN rules r ON rt.id = r.id WHERE r.enabled=1";
cur.execute(stmt)
storedRules = cur.fetchall()

# Join them
rulesText = ";";
for rule in storedRules:
    rulesText += rule[0] + '\n'

# Load them into yara
rules = yara.compile(source=rulesText)


if len(sys.argv) > 1:
    scanFile(int(sys.argv[1]))

# RabbitMQ callback
def callback(ch, method, properties, body):
    print "; [x] Received %r"; % (body,)
    scanFile(int(body))
    ch.basic_ack(delivery_tag = method.delivery_tag)

# Consume messages from work queue
connection = pika.BlockingConnection(pika.ConnectionParameters(
               'localhost'))
channel = connection.channel()
channel.queue_declare(queue='uploaded_files')
channel.basic_consume(callback,
                      queue='uploaded_files')

print ' [*] Waiting for messages. To exit press CTRL+C'
channel.start_consuming()
{% endhighlight %}

<h3>Getting results</h3>
At this point, a user can upload a file and it will get scanned and hashed, but they won't be able to see any of the results, so let's work on our web server a little to give some feedback.  We'll just create some handlers that can show them the database data as json output.

<!-- highlight="10,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,33,34,35,36,37,38,39,40,41,42,43,44,45,46,48,49,50,51,52,53,54,55,85,86,87" -->

{% highlight python linenos=table %}
#!/usr/bin/python
# file: webserver.py

import tornado.ioloop
import tornado.web
import tornado.escape

import utils
import pika
from datetime import datetime

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render("static/index.htm")


class GetFilesHandler(tornado.web.RequestHandler):
    def get(self):
        stmt = "SELECT id, submission_date, filename, size, md5 from files ORDER BY id DESC LIMIT 10"
        cur.execute(stmt)
        files = cur.fetchall()
        output = []
        for f in files:
            output.append({
                'id': f[0],
                'submission_date': f[1].strftime('%Y-%m-%d %H:%M:%S'),
                'filename': f[2],
                'size': f[3],
                'md5': f[4],
                })
        self.write(tornado.escape.json_encode(output))
        self.finish()


class GetMatchesHandler(tornado.web.RequestHandler):
    def get(self, file_id):
        stmt = "SELECT rule_id, description from matches m join rules r on m.rule_id = r.id where m.file_id = %(file_id)s"
        cur.execute(stmt, {'file_id': file_id})
        matches = cur.fetchall()
        output = []
        for m in matches:
            print m
            output.append({
                'rule_id': m[0],
                'description': m[1]
                })
        self.write(tornado.escape.json_encode(output))
        self.finish()


class GetRuleHandler(tornado.web.RequestHandler):
    def get(self, rule_id):
        stmt = "SELECT text from rules_text WHERE id = %(id)s"
        cur.execute(stmt, {'id': rule_id})
        rule = cur.fetchone()[0]
        self.write(rule)
        self.set_header('Content-Type', 'text/plain')
        self.finish()


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

        # Queue work message
        connection = pika.BlockingConnection(pika.ConnectionParameters(
                       'localhost'))
        channel = connection.channel()
        channel.queue_declare(queue='uploaded_files')
        channel.basic_publish(exchange='',
                      routing_key='uploaded_files',
                      body='%s' % file_id)
        connection.close()


cur, db = utils.connectToDB()

application = tornado.web.Application([
    (r"/file-upload", UploadHandler),
    (r"/getFiles", GetFilesHandler),
    (r"/getMatches/([0-9]+)", GetMatchesHandler),
    (r"/getRule/([0-9]+)", GetRuleHandler),
    (r"/", MainHandler),
    (r'/(.*)', tornado.web.StaticFileHandler, {'path': 'static'}),
])

if __name__ == "__main__":
    application.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
{% endhighlight %}

<h3>Conclusion</h3>
Now our user can upload a file and by typing in specific URL's they can see the results, but it's a really sloppy UI.  Next, we'll go over how to make all this data visible to the user.  Also, we need to do some final clean-up so we don't have to manually start these services.

Next <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-5-of-5-finishing-touches/">Part 5</a>
