---
layout: post
title: Root Certificate Authority research
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1357712709'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
<b>Update 2013.01.07</b>: asteriskpound on <a href="http://www.reddit.com/r/netsec/comments/162rxw/root_certificate_authority_research/">reddit</a> has pointed out a flaw in how I determine the root certifcate, and how I calculate the length of the certificate.  The flaw is that I thought that the last certificate in the "certificate chain" from openssl's output would always be the root of the chain, but actually this "chain" can be very broken (as is the case with me thinking www.olivenoel.com had 21 certificates in it's chain).  I don't expect my final results to be very different, but I will need to re-evaluate before these results can be trusted.

<b>Update 2013.01.09</b>: See my follow-up post <a href="http://0xdabbad00.com/2013/01/08/ca-research-post-2/">Root Certificate Authority research – post 2</a> for corrections to the data.

<h3>Highlights</h3>
Only 50 of the top 1M sites (from <a href="http://www.alexa.com/">Alexa</a>) are signed by TURKTRUST. For the conspiracy theorists out there, 22 of those are for Iranian sites, with the rest being for Turkish sites.  See the list of those sites at: <a href="http://pastebin.com/PdKb5BkF">http://pastebin.com/PdKb5BkF</a> (site name followed by certificate info).

22% of the top 1M sites have an SSL site (not using a self-signed cert, but possibly not all of these are valid).  86% of all SSL sites are signed by only 20 root certificates, and because many companies (such as Verisign) have multiple root certificates, 98% of all SSL sites are signed by 20 companies.  For counts on the top 1000 root CAs for the top 1M sites, see:
<a href="http://pastebin.com/kgd1g2m3">http://pastebin.com/kgd1g2m3</a>

To look at this data yourself, you can download the file <a href="https://www.dropbox.com/s/phc1pvafum44nd8/certPaths.zip">certPaths.zip (22MB)</a>.  The format (further shown in the Appendix) is: <tt>site|# of certs in chain to root|cert info|issuer info|root info</tt>

<h3>Motivation and research</h3>
Before I get into how I gathered this data, let me explain what provoked me to do this.   Due to the recent <a href="http://www.h-online.com/security/news/item/Fatal-error-leads-TURKTRUST-to-issue-dangerous-SSL-certificates-1777291.html">TURKTRUST news</a>, and similar <a href="http://www.h-online.com/open/news/item/Fake-Google-certificate-is-the-result-of-a-hack-1333728.html">Diginotar news</a> from 2011, involving root Certificate Authority (CA) mistakes, I decided to do some research into certificate authorities this weekend.  The basic problem is that all browsers trust a bunch of weird root certificate authorities, and these CA's can create certs for any site, which your browser will trust without any warnings or indications if one of these CA's does something weird (like TURKTRUST signing google.com certs).  So although the cert for www.google.com is signed by Verisign, there are no technical restrictions for Turktrust or Diginotar to have signed a different cert to allow a server to pretend to be the real www.google.com.  Verisign and Turktrust are each trusted equally by your browser (and Diginotar used to be).  So if someone can get one of these fraudulent certs and MiTM your browser, then when you connect to gmail, they can read all your traffic as if you weren't using SSL at all.

There has been various research into trying to improve this situation.  The <a href="https://www.eff.org/observatory">EFF SSL Observatory</a> project has tried to identify how many CA's there really are that your browser trusts, and has identified <b>1,482 CAs trusted by Microsoft and Mozilla</b> from 651 organizations (many organizations have multiple certs).  Even though your browser may "only" trust 194 CA's (as is the case with my Firefox 17.0.1 browser as of Jan 6, 2013), those CA's have signed intermediate CA's and given them the ability to sign more certs.  Furthermore, although IE may only show 30 CA's that it trusts (as is the case with my Internet Explorer 9.0.9112.16421 install), it actually trusts many more which Microsoft downloads and trusts on an as-needed basis when it runs into them as explained in the technet article <a href="http://technet.microsoft.com/en-us/library/cc751157.aspx">Microsoft Root Certificate Program</a> (pointed out in "Certified Lies" paper discussed later in this post).  Chrome relies on the OS to provide its trust, so on my Windows 7 system it has the 30 CA's that IE does, and on my Ubuntu system it trusts 145 certificates.

It is important to note that not any cert can create and sign another cert.  That issue has been talked about by Moxie Marlinspike in his <a href="http://www.thoughtcrime.org/software/sslstrip/">2009 Blackhat DC presentation on sslstrip</a>, and should not happen with current browsers.

However, a default browser install today still trusts many more organizations than many believe it should, and those CAs can have intermediate CAs which are capable of signing certificates that do not show up as root CAs in the browsers. There is a great paper called <a href="cryptome.org/ssl-mitm.pdf">"Certified Lies: Detecting and Defeating Government Interception Attacks Against SSL"</a> by Christopher Soghoian and Sid Stamm, explaining how SSL can be exploited due to these trust issues, and introduces their Firefox addon <a href="https://code.google.com/p/certlock/">CertLock</a> which implements their Trust-On-First-Use (TOFU) philosophy wherein you trust the Google cert the first time you see it, and panic if it ever changes.  There is a post on the <a href="https://blog.torproject.org/blog/life-without-ca">TOR project</a> about a guy that removes all certs and adds them as needed, and another <a href="http://netsekure.org/2010/05/results-after-30-days-of-almost-no-trusted-cas/">post</a> shows the 10 CA's that were ultimately accepted by someone after 30 days of using a browser with which he had originally removed all CA's and only added them as needed.

<h3>How to protect yourself</h3>
So what should you do?  Well, first, SSL is still a good thing, and you should still use it as much as possible.  To ensure you do, <b>install EFF's <a href="https://www.eff.org/https-everywhere">HTTPS Everywhere</a></b> browser add-on which will try to force sites to use SSL that offer it.  There are a few additional extensions (all for Firefox only) which have sought to fix this particular problem (I have not tested these):
<ul>
<li><a href="https://code.google.com/p/certlock/">CertLock</a> the project discussed previously to use the Trust-On-First-Use philosophy.
<li><a href="https://addons.mozilla.org/en-US/firefox/addon/perspectives/">Perspectives</a> explained on their site <a href="http://perspectives-project.org/">perspectives-project.org</a> that they provide 3rd party notaries to check with to ensure a cert should be trusted, similar to the "web of trust" concept used by PGP.
<li><a href="https://addons.mozilla.org/en-us/firefox/addon/certificate-patrol/">Certificate Patrol</a> reveals when certificates are updated, so you can ensure it was a legitimate change.
</ul>

<h3>My research</h3>
<h4>How many CA's do my browsers trust?</h4>
My question in all this was "Why do I need to trust so many CA's?"  So I decided to first see how many I trust.  I manually exported all the CA's in Firefox and IE (Chrome on Windows has the same list as IE as they both get them from the OS apparently).  This was a pain, so I've made these files available at:
<ul>
<li><a href="https://www.dropbox.com/s/uv1gbrlu08h83dp/firefox_ca_certs.zip">firefox_ca_certs.zip</a> (194 CA's)
<li><a href="https://www.dropbox.com/s/yogrvjkb0hbe98s/ie_ca_certs.zip">ie_ca_certs.zip</a> (30 CA's)
</ul>

All certs are in the PEM format and can be viewed using:
<pre>openssl x509 -in certum.pem -text -noout</pre>

This allows you to see information such as:
<pre>
Signature Algorithm: sha1WithRSAEncryption
Issuer: C=PL, O=Unizeto Sp. z o.o., CN=Certum CA
Validity
  Not Before: Jun 11 10:46:39 2002 GMT
  Not After : Jun 11 10:46:39 2027 GMT
</pre>

If you would like to export your own files from Chrome or IE, you will need to covert them from the DER format to PEM using:
<pre>openssl x509 -inform der -in certificate.cer -out certificate.pem</pre>

<h4>How many CA's do I need to trust?</h4>
I wanted to see how many CA's are really in use, so I downloaded the <a href="http://s3.amazonaws.com/alexa-static/top-1m.csv.zip">Alexa Top 1 Million sites</a> in the world, which just gives a list like this:
<pre>
1,facebook.com
2,google.com
3,youtube.com
4,yahoo.com
5,baidu.com
</pre>

I then wrote some scripts to collect certificate info using openssl (see Appendix for scripts).

<h3>Other interesting results</h3>
Not really relevant, but I found it interesting to see how many certs were in the chain to the root.
<table>
<tr><th># of certs in chain<th>Site count</tr>
<tr><td>0 (site did not have ssl)<td>640734</tr>
<tr><td>1 (self-signed)<td>123733</tr>
<tr><td>2<td>100136</tr>
<tr><td>3<td>95165</tr>
<tr><td>4<td>26648</tr>
<tr><td>5<td>1561</tr>
<tr><td>6<td>97</tr>
<tr><td>7<td>49</tr>
<tr><td>8<td>18</tr>
<tr><td>9<td>25</tr>
<tr><td>10<td>1</tr>
<tr><td>11<td>6</tr>
<tr><td>12<td>3</tr>
<tr><td>13<td>55</tr>
<tr><td>14<td>1</tr>
<tr><td>15<td>2</tr>
<tr><td>16<td>1</tr>
<tr><td>17<td>12</tr>
<tr><td>18<td>2</tr>
<tr><td>19<td>5</tr>
<tr><td>21<td>1 www.olivenoel.com</tr>
</table>

<h3>Appendix</h3>
To grab cert info from the Alexa Top 1 Million sites list, I create a bash script called <b><tt>output_cert_info.sh</tt></b> with the following:
[sourcecode language="bash" gutter="false"]
#!/bin/sh
echo $1
timeout 3 openssl s_client -connect $1:443 &lt; /dev/null &gt; data/$1 2&gt;/dev/null

echo &quot;HEAD / HTTP/1.0
Host: $1:443

EOT
&quot; \
| timeout 3 openssl s_client -connect $1:443 2&gt;/dev/null \
| sed -n '/-----BEGIN CERTIFICATE-----/,/-----END CERTIFICATE-----/p' \
| openssl x509 -noout -text -certopt no_signame &gt; x509/$1 2&gt;/dev/null
[/sourcecode]

and ran it with the following:
[sourcecode language="bash" gutter="false"]
cat top-1m.csv | sed 's/.*,/www./g' | xargs -P50 -I {} ./output_cert_info.sh {}
# Takes about 16 hours
[/sourcecode]

That turns all the sites into www. names, and using some <a href="http://web.archive.org/web/20110520140532/http://teddziuba.com/2010/10/taco-bell-programming.html">Taco Bell Programming</a> to parallelize this.  The bash script itself will record data about the certificate chain to the <tt>./data</tt> dir and the x509 cert itself to <tt>./x509</tt>.

To extract the certificate path info from the data directory, I wrote a python script called <b><tt>gatherData.py</tt></b>:
[sourcecode language="python" gutter="false"]
import sys
import os
import re

for filename in sys.stdin:
	filename = filename.strip()
	site = os.path.basename(filename)
	try:
		with open(filename) as f:
		    content = f.readlines()
		    certData = False
		    certNum = 0
		    isIssuer = False
		    subject = &quot;&quot;
		    issuer = &quot;&quot;
		    root = &quot;&quot;
		    for line in content:
		    	line = line.strip()

		    	if line == &quot;Certificate chain&quot;:
		    		certData = True
		    		continue
		    	if certData:
		    		if line == &quot;---&quot;:
		    			certData = False
		    			break
		    		if not isIssuer:
		    			if subject == &quot;&quot;:
		    				subject = re.sub(&quot;%d s:&quot; % certNum, &quot;&quot;, line)
		    			certNum += 1
		    			isIssuer = not isIssuer
		    			continue
		    		if isIssuer:
		    			root = re.sub(&quot;i:&quot;, &quot;&quot;, line)
		    			if issuer == &quot;&quot;:
		    				issuer = root
		    			isIssuer = not isIssuer
		    			continue
		    print &quot;%s|%d|%s|%s|%s&quot; % (site, certNum, subject, issuer, root)
		    sys.stdout.flush()
	except:
		print &quot;%s|-1|||&quot; % (site)
[/sourcecode]

I ran this with:
[sourcecode language="bash" gutter="false"]
cd data
ls -f | python ../gather_data.py &gt; ../certPaths
# Takes about an hour
[/sourcecode]

The generated file (<tt>certPaths</tt>) is only 988257 lines long, instead of 1 million, because the original list has a lot of non-site lines, such as maharojgar.gov.in/~selfemp.

The <tt>data</tt> directory is 2.1GB uncompressed, so I have not posted it online, but can make it and the <tt>x509</tt> directory available on request.
