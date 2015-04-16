---
layout: post
title: ! 'File scanner web app (Part 3 of 5): YARA signatures'
categories: []
tags: []
status: publish
type: post
published: true
---
<a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-1-of-5-stand-up-and-webserver/">Part 1</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-2-of-5-upload-files/">Part 2</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-3-of-5-yara-signatures/">Part 3</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-4-of-5-scanning-files-from-the-web-app/">Part 4</a>, <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-5-of-5-finishing-touches/">Part 5</a>

So far in this training series, we've created a web app in a VM that is capable of having files uploaded to it.   Now we're going to take some time away from our web app to play with <a href="https://code.google.com/p/yara-project/">YARA</a>.  YARA is basically just grep for binary files, and catered specifically to executables for malware identification.  It can just as easily scan PDF files or any other file type, and it can do things beyond just looking for malware signatures.  You use YARA to look for specific byte patterns of interest, which may indicate that it is similar to other files you've analyzed previously.  Black-listing, like what AV products do, is dying as a protection capability, but scanning for byte signatures will always have a place in providing some insight into otherwise unknown files.




<h3>Install YARA</h3>
First, on the guest, we need to get the development tools and some required libraries, then we'll download and build YARA.
{% highlight text %}
sudo apt-get install build-essential python-dev libpcre3 libpcre3-dev unzip
cd /tmp
wget https://yara-project.googlecode.com/files/yara-1.7.tar.gz
tar -zxvf yara-1.7.tar.gz
cd yara-1.7
./configure
make
sudo make install
cd ..
yara -v # Should print: yara 1.7 (rev:167)
wget https://yara-project.googlecode.com/files/yara-python-1.7.tar.gz
tar -zxvf yara-python-1.7.tar.gz
cd yara-python-1.7
python setup.py build
sudo python setup.py install
cd ..
rm -rf yara-1.7*
rm -rf yara-python-1.7*
{% endhighlight %}

As root, we need to do the following for the YARA python library to work:
{% highlight text %}
sudo su -
echo "/usr/local/lib" >> /etc/ld.so.conf
ldconfig
exit
{% endhighlight %}

Now run <tt>python</tt> and try <tt>import yara</tt>.  It should not show any errors.

<h3>Test out YARA</h3>
Let's create the following rule file (<tt>yara_rules.txt</tt>):

{% highlight text %}
rule IsPE
{
	meta:                                        
        description = "Windows executable file"

	condition:
		// MZ signature at offset 0 and ...
		uint16(0) == 0x5A4D and
		// ... PE signature at offset stored in MZ header at 0x3C
		uint32(uint32(0x3C)) == 0x00004550
}

rule has_no_DEP
{
	meta:
        description = "DEP is not enabled"

	condition:
		IsPE and
		uint16(uint32(0x3C)+0x5E) & 0x00100 == 0
}

rule has_no_ASLR
{
	meta:
        description = "ASLR is not enabled"

	condition:
		IsPE and
		uint16(uint32(0x3C)+0x5E) & 0x0040 == 0
}
{% endhighlight %}

If you're not familiar with the PE file structure, check out my projects <a href="http://icebuddha.com/index.htm?test=1">IceBuddha</a> and <a href="http://icebuddha.com/slopfinder.htm">SlopFinder</a>.

Now let's get some Windows executables and try it out.
{% highlight text %}
wget http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe
wget http://download.sysinternals.com/files/Autoruns.zip
unzip Autoruns.zip
yara yara_rules.txt putty.exe # Should show all 3 rules
yara yara_rules.txt autoruns.exe # Should show only IsPE because it has DEP and ASLR
{% endhighlight %}

<h3>Create MySQL tables</h3>
Keeping track of rules is not an important part of this tutorial unfortunately, so I'm going to be lazy here.  Feel free to extend this project by adding proper abilities to edit rules.

Connect the mysql client with <tt>mysql -u root -p scanner</tt> and enter the password, then create tables as follows:
{% highlight text %}
create table rules (
  id INT NOT NULL AUTO_INCREMENT,
  submission_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  name VARCHAR(30) NOT NULL DEFAULT "",
  description VARCHAR(200) NOT NULL DEFAULT "",
  enabled TINYINT NOT NULL DEFAULT 1,
  PRIMARY KEY(id)
);

create table rules_text (
  id INT NOT NULL,
  text VARCHAR(1024),
  PRIMARY KEY(id)
);

create table matches (
  file_id INT NOT NULL,
  rule_id INT NOT NULL,
  PRIMARY KEY(file_id, rule_id)
);
{% endhighlight %}

 The <tt>rules</tt> table will keep track of the names of our rules and description field, with the rules themselves being stored in <tt>rules_text</tt>.  The matches table will allow us to correlate rules that matched to the files they matched against.

<h3>Add rules</h3>
Like I said, this is really sloppy code.  I'm just going to disable any previous rules and add my current set of rules, and I'm going to do some poor man's parsing to extract out the rules and metadata, meaning don't create rules that contain words like 'rules' or 'description' or this will break.

First, I create a <tt>utils.py</tt> file which we'll use to set up our DB connection so we don't need to store our credentials in every file.
{% highlight python %}
#!/usr/bin/python
# file: utils.py

import MySQLdb

def connectToDB():
	db = MySQLdb.connect(host="localhost",
	                     user="root",
	                      passwd="mypassword",
	                      db="scanner")
	cur = db.cursor()
	return cur, db
{% endhighlight %}

Next, we have <tt>updaterules.py</tt> for adding the rules.

{% highlight python %}
#!/usr/bin/python
# file: updaterules.py

import utils
import sys

if len(sys.argv) < 2:
	print "Usage: %s <rules_file>" % sys.argv[0]
	sys.exit(0)

rulesFile = sys.argv[1]
print "Loading rules from %s" % rulesFile

# Read the rules file
with open(rulesFile) as f:
	rulesData = f.read()

# Extract out each rule
rule = {'name': '', 'description': '', 'text': ''}
rules = []
for line in rulesData.split('\n'):
	if line.startswith('rule'):
		# Add previous rule to list
		if rule['text'] != '':
			rules.append(rule)
			rule = {'name': '', 'description': '', 'text': ''}
		rule['name'] = line.replace('rule', '').strip()
	if line.strip().startswith('description'):
		rule['description'] = line.replace('description =', '').replace('\"', '').strip()
	rule['text'] += line + '\n'
rules.append(rule)

cur, db = utils.connectToDB()

# Disable all previous rules
stmt = "UPDATE rules set enabled = 0"
cur.execute(stmt)
db.commit()

# Add our new rules
for rule in rules:
	stmt = "INSERT INTO rules (name, description) VALUES (%(name)s, %(description)s)"
	cur.execute(stmt, {'name': rule['name'], 'description': rule['description']})
	rule_id = cur.lastrowid
	db.commit()
	
	stmt = "INSERT INTO rules_text (id, text) VALUES (%(id)s, %(text)s)"
	cur.execute(stmt, {'id': rule_id, 'text': rule['text']})
	db.commit()

	print "Added rule %s" % rule['name']

db.close()
print "Complete"
{% endhighlight %}

As sloppy as this is, we can now simply run the following to add our rules:
{% highlight text %}
/var/apps/scanner$ python updaterules.py yara_rules.txt 
Loading rules from yara_rules.txt
Added rule IsPE
Added rule has_no_DEP
Added rule has_no_ASLR
Complete
{% endhighlight %}

<h3>Scanning files with YARA through Python</h3>
Now we'll create a python script that can take a <tt>file_id</tt> as an argument.  It will scan the file in the <tt>uploads</tt> directory using the rules stored in the database and then record in the database any matches it finds in the <tt>matches</tt> table.

{% highlight python %}
#!/usr/bin/python
# file: yarascanner.py

import utils
import yara
import sys
from os import path

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


cur, db = utils.connectToDB()

stmt = "SELECT text FROM rules_text rt JOIN rules r ON rt.id = r.id WHERE r.enabled=1"
cur.execute(stmt)
storedRules = cur.fetchall()

# Join them
rulesText = ""
for rule in storedRules:
    rulesText += rule[0] + '\n'

rules = yara.compile(source=rulesText)


if len(sys.argv) > 1:
    scanFile(int(sys.argv[1]))
{% endhighlight %}

Running this over our two files (using something like <tt>python yarascanner.py 4</tt>) should result in something like the following in the database:
{% highlight text %}
mysql> select * from matches;
+---------+---------+
| file_id | rule_id |
+---------+---------+
|       4 |       4 |
|       5 |       4 |
|       5 |       5 |
|       5 |       6 |
+---------+---------+
4 rows in set (0.00 sec)
{% endhighlight %}

We can now insert rules into our database (using <tt>updaterules.py</tt>) and scan files using those rules (using <tt>yarascanner.py</tt>) which will result in our database containing the matches found.

Next <a href="http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-4-of-5-scanning-files-from-the-web-app/">Part 4</a>
