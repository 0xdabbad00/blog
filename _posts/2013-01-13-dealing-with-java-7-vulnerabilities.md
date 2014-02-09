---
layout: post
title: Dealing with Java 7 vulnerabilities
categories: []
tags: []
status: publish
type: post
published: true
---
This post is mostly just organizing all the info that's been posted lately about Java 7 problems.

<b>Update 2013-01-13</b>: Update now available from Oracle for <a href="https://www.java.com/en/download/index.jsp">Java 7 Update 11</a> to fix this vuln.

<h3>Defining the current problem</h3>
There is a <a href="http://labs.alienvault.com/labs/index.php/2013/new-year-new-java-zeroday/">Java 0-day</a> going around (<a href="https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-0422">CVE-2013-0422</a> the one discovered by <a href="https://twitter.com/kafeine">@Kafeine</a>).  Java was once viewed as being the most secure language to write code in because many common memory corruption vulnerabilities are difficult (impossible?) to create.  Lately, however, Java is getting a bad reputation due to it's growing list of vulnerabilities, and slow reactions from Oracle, and in the case of this bug, it improperly patched the problem.  

Many security professional are advocating the naive binary reaction "Java is bad, get rid of Java now!"  Really?  You're being paid 6-figures to give the response of a 6-year-old?  This is not a business reality.  We can't command every user out there to not use a product until this storm blows over.  Because Java historically has been viewed as the option of choice for those security minded developers, many applications have been written in it. For the home user, they likely encounter Java in their banking site. For the enterprise user, there are likely using Java applications to do their jobs.  You might as well advocate "If your company made best practice decisions in the past and wrote many applications in Java, which you use to do your job, take a vacation until a patch comes out, or rewrite all your applications to not use it.  I changed my mind about Java being secure."  These are the problems we're supposed to assist the world in solving.  I'm in this profession to help make computers secure, because I use them, I believe the world is better because of them, and I want them to be secure.

First, let's understand the problem.  To really dig into the guts of it, here are some good resources:
<ul>
<li>Bugtraq <a href="http://seclists.org/bugtraq/2013/Jan/48">[SE-2012-01] 'Fix' for Issue 32 exploited by new Java 0-day code</a>
<li>stopmalvertising.com <a href="http://stopmalvertising.com/malware-reports/analysis-of-cve-2013-0422-a-new-0-day-java-exploit.html">Analysis of CVE-2013-0422 - A New 0-day Java Exploit</a>
<li>immunity.com <a href="https://partners.immunityinc.com/idocs/Java%20MBeanInstantiator.findClass%200day%20Analysis.pdf">MBeanInstantiator.findClass 0-day Analysis.pdf</a>
<li>General problems in Java 7 from security-explorations.com in their doc <a href="http://www.security-explorations.com/materials/se-2012-01-report.pdf">se-2012-01-report.pdf</a>
</ul>

<h3>Current solutions</h3>
Let's explore the options to handle this vulnerability.  The nuclear option is to uninstall Java completely (check the directories `C:\Program Files\Java\` or `C:\Program Files (x86)\Java\`).  The basic problem though is that Java runs in the browser and this is where the trouble happens, so if you still need Java, but not in the browser, just disable Java in the browser.  If you have a Java app that runs locally on your system, it's likely not going to be affected by this vulnerability.  You're only vulnerable if you are executing arbitrary Java files, which is what the browser does.  Even if you don't go to sketchy sites, those sites could have been hacked, or they could have advertisements which bad guys can "advertise" things on which execute Java.

<h4>Disable Java in the browser</h4>
If you use IE, you can't really disable Java from the browser, so if possible use Mozilla or Chrome.  If you use Mozilla or Chrome, those browser now force you to "Click to play" the java code, effectively stopping the exploit unless you click on it.  Interestingly, Apple took the approach of <a href="http://www.h-online.com/security/news/item/Java-plugins-unplugged-by-Mozilla-and-Apple-1782628.html">black-listing Java entirely</a>.  <a href="https://krebsonsecurity.com/how-to-unplug-java-from-the-browser/">Krebs on Security</a> discusses how to disable Java on various browsers, in case the "Click to play" solution is not enough for you. To check if your browser is still running Java, go to <a href="https://www.java.com/en/download/testjava.jsp">https://www.java.com/en/download/testjava.jsp</a>.

<h4>Restricting Java in the browser</h4>
If you still need Java in the browser, you can restrict it to only certain sites as described on <a href="http://www.darkoperator.com/blog/2013/1/12/pushing-security-configuration-for-java-7-update-10-via-gpo.html">darkoperator.com</a>.  This may still have <a href="https://en.wikipedia.org/wiki/Cross-site_scripting">XSS</a> issues.  Or alternatively, for Firefox and Chrome (on Linux or OSX only) you can use <a href="https://code.google.com/p/nssecurity/">nssecurity</a>.

<h4>Patch</h4>
If you'd like to patch this problem, you can get an unofficial patch from <a href="http://schierlm.users.sourceforge.net/CVE-2013-0422.html">http://schierlm.users.sourceforge.net/CVE-2013-0422.html</a>.  This adds some additional checking code to vulnerable code to disable it.  Unfortunately, things might be in a weird state when the next Java update comes along.

<h3>Desired future solutions</h3>
<h4>White-list signed jars</h4>
Java has for a long time come with the ability to sign jar files, but this only allows for additional privileges to be added to signed code.  See the post <a href="http://securesoftwaredev.com/2012/11/12/sandboxing-java-code/">Sandboxing Java Code</a> on securesoftwaredev.com.  I would like to have the ability to say "Only code that has been signed by the following entities can run at all".  I don't know of any way of currently making that possible.

<h4>Reduce the Java Attack Surface</h4>
Just as you can patch the JRE, you can also just remove entire classes.  A lot of Java problems are due to it's Reflection library, so you could just remove those.  Java provides functionality to play sounds and graphics libraries for games.  Java provides a lot of backwards functionality to use old, out-dated crypto libraries.  If I just need Java for business uses, I can strip the JRE of all this additional functionality.  However, it's a pain.  There should be an easier way.  If you run java with the <tt>verbose:class</tt> flag, you can see the classes as they are loaded.  Someone ought to write a tool to hook that, and provide a way to disable most of the JRE, and be able tell the user "This class tried to access restricted class X, so it was killed."  Obviously, this is much more technical than is useful for the average user, but I think such a tool should probably exist.
