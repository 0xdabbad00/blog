---
layout: post
title: Harden OSX and iPhone for security/hacking conventions (Defcon)
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1342483160'
  _edit_last: '2'
---
<b>tl;dr</b>: No brilliant ideas here, just a check-list of basic steps.

Defcon is coming up.  Time to actually secure all my toys/tools.  Ideally, you would bring no electronics with you, and wrap yourself in tin foil, but a day without Internets is a day wasted, so the next alternative is you'd bring an old laptop you don't care about, wipe it, fresh install of a BSD distro (since those are smaller targets with less surface area), update, lock it down, and wipe it when you get home.  You'd connect to the Internet via something other than wifi through a VPN.  Your phone would be an old dumb phone with no contacts in it and you'd carry around a paper address book.

However, you can't let your fear of being owned hold you hostage, so let's go through how to attend Defcon and at least take the minimum steps to bring the latest and greatest toys with you.  I also haven't exactly bothered yet to see how to lock down my toys as I am new to OSX, and I hadn't really bothered to look at my iPhone's security much yet.

<h3>Concerns</h3>
People get concerned about defcon because it's a giant gathering of hackers, and hackers hack, but is your risk really any higher at defcon?  On the one hand, 0-days against Windows or anything else can be sold for a lot of money, and if someone started trying to use one at defcon, there are enough smart people there that someone might sniff it, or run a honeypot to catch it, and it would quickly no longer be 0-day, and lose it's value.  So they'd need a pretty good reason to use it, but considering many of the people at the convention are going to be good targets for many others there, this might be an acceptable risk.  Consider that a blackhat would very much like to be able to read the email going to security@microsoft.com in order to find out about new 0-days, or if you were committing cyber crimes, wouldn't you want to read the emails of people that go after cyber criminals to find out if they were coming after you?

The main things to worry about are that all your network traffic is going to be sniffed and you very much so will be exposed to MiTM attacks, even if they are as "harmless" as having images your browser receives replaced by goatse images.  So make sure all your network traffic is encrypted, by going to SSL sites and using a VPN. 

<h3>Basic strategy</h3>
<ul>
<li>Up-date everything and reduce attack surface area (basic security)
<li>Remove as much data as possible before you go, and refresh when you get back.
</ul>

<h3>Macbook Air - OSX 10.7</h3>
<ol>
<li>Under System Preferences:
<ol>
<li>Bluetooth: Turn off (Default is on)
<li>Network: Ensure "Ask to join new networks" is checked.
</ol>
<li>Download and install nmap (don't install the updater), and run: nmap -v -sT 192.0.0.1 (replace with your ip).  Ensure no ports are open.
<li>Backup system. 
<ol>
<li>I was not able to boot from a Knoppix CD (plugged in an external CD/DVD drive), so I made a bootable USB key with Knoppix on it.  To do so, use the the instructions here: <a href="http://knoppix.net/wiki/Bootable_USB_Key">http://knoppix.net/wiki/Bootable_USB_Key</a>.  This is more painful than it should be (I couldn't find just a .iso to burn to the thumb-drive anywhere).
<li>Install <a href="http://refit.sourceforge.net/">rEFIt</a> on macbook.
<li>Plug-in Knoppix usb key and reboot macbook multiple times (at least 2) until the rEFIt screen gives you the option to boot from it.  When it does, choose it, and boot into knoppix.
<li>From command-line, as root, make dd image (something like: dd if=/dev/sda of=/media/sdc1/backup.img)
</ol>
<li>Use VPN when connected to the Internet.  Try to avoid any wifi near the Defcon convention.  When choosing a VPN to use, consider that if you set-up a VPN on your home system that system may then become a target, so use your company's (pros/cons there too I guess), set one up on an Amazon AWS instance that you can throw away, or use a VPN service, although the VPN service options seem sketchy to me (some VPN providers want to use their own special VPN clients, such as a "modified" OpenVPN in the case of one I saw).
<li>When you get back:
<ol>
<li>Copy the image back to the laptop following the reverse process.
<li>Change passwords.
</ol>
</ol>

<h3>iPhone 4S</h3>
At defcon, I'm going to only use my laptop for data.  So I'll disable everything except voice.  Start in Settings -> General:
<ol>
<li>Bluetooth: Off
<li>Software Update: Ensure up-to-date
<li>Network: Cellular Data: Off; Wifi: Off
<li>Passcode lock: On (In case someone physically steals your iPhone).
<li>Settings->Location Services: Off
</ol>

I'm not sure how to image an iPhone, but it would require jail-breaking it.  I'm not going to try to figure out how to image it.

If you do use your iPhone for the Internet, don't use wifi.  Clear history and cookies, and disable javascript.
