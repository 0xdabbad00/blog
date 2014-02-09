---
layout: post
title: System Engineering
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1325281074'
  _edit_last: '2'
---
Recently, I started working on a side project relevant to finance.  People don't want to follow links to finance sites from sites related to malware, so I don't want to be specific about it.  Anyway, I figured I'd host it here on Hostgator where <a href="0xdabbad00.com">0xdabbad00.com</a> is hosted.  Hostgator has been good, with decent support, but more importantly I get ssh access to the server and it runs a mysql instance.  For this finance site, I needed to have a database with 15M rows of data, consisting of about 1GB of data.  Unfortunately, HostGator restricts any "query" to it's MySQL to a few seconds, including bulk imports, so it became quite frustrating to work with.

I decided to switch to Google App Engine, because it is free, and the cool kids think it's cool so I wanted to check it out.  Getting the files for the site up was a little more painful than it needs to be.  You don't have any ssh access or normal way of uploading the files.  You run some python scripts and describe the purpose of your files using YAML.  The painful part was trying to get the database working.  Everything about Google's GQL and working with BigTable through Google App Engine (GAE) is more painful than I feel it should be.  Ultimately though, I ran into a deal breaker: GAE only allows 100K DB writes/day, and I had 15M rows I needed loaded.  My options were pay ($15 I'm guessing), upload a little each day, or make some sloppy "fake" database using files.  Those options are all bad, so I switched to Amazon Web Services.  GAE has it's purpose, but just not for my needs.

Amazon Web Services (AWS) gives you access to a lot of things in the first year all for free, and it's clear right from start that AWS is targeted to large complex systems with tons of computers that all need to talk with one another.  For my needs though, I just needed to get a single compute instance running on EC2, install a LAMP stack on it, upload my files and set up the DB.  Since you are basically ssh'ing into a VM, once you have the EC2 instance up, there is no learning curve.

Once my free period with AWS is over, I might hate them, because I hear they are expensive, but for now, I think they are awesome.  I don't have to worry about hardly any system engineering work.  System Engineering has come along way recently.  I'm starting to get really interested in <a href="http://vagrantup.com/">Vagrant</a> for setting up a system rapidly.  Check out this post from Mozilla on <a href="http://blog.mozilla.com/webdev/2011/10/04/developing-with-vagrant-puppet-and-playdoh/">Developing with Vagrant, Puppet, and playdoh</a> which explains why this is cool.

I'm even starting to get a little interested in monitoring due to things like Google Analytics which I run on the other site.  I don't have ads or any tracking pixels or iframes or anything on this site because I don't like them.  I run Adblock Plus and Ghostery on every browser.  I don't even understand how people can use a browser without Adblock Plus installed.  The internet looks horrible on other people's browsers with all of it's ads.  Also, I fully expect all of the visitors to this site to have Adblock installed so there wouldn't be any point of me putting ads on here anyway.  The point of this site is for me to babble about whatever interests me and every once in a while people contact me with interesting stuff because of it.  The point of my other site is quite simply to make money through ads and maybe something more useful one day.
