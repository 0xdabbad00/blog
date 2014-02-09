---
layout: post
title: Root Certificate Authority research - post 2
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1357682300'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
In my last post, "<a href="http://0xdabbad00.com/2013/01/06/root-certificate-authority-research/">Root Certificate Authority research</a>", I had attempted to identify the certificate chains of the top 1 million sites, as identified by <a href="http://www.alexa.com/">Alexa</a>.  My data collection method was flawed, so this time around, I hope to fix that.

<h3>Collecting data</h3>
Previously, I had taken the Alexa Top 1M list, prefixed "www" to all the domains, and tried to contact the servers on port 443.  That yielded 347521 ssl capable sites.  Using that, I then ran it through this script:
[sourcecode language="bash" gutter="false"]
#!/bin/sh
# Filename: ./get_cert_chains.sh
# Run as:
#cat ssl_sites | xargs -P10 -I {} ./get_cert_chains.sh {} 2&gt;cert_chains/$1 &gt; /dev/null
# (I have 8 cores, but some sites stall out, so I parallelized to 10)

echo $1
echo &quot;HEAD / HTTP/1.0
Host: $1:443

EOT
&quot; \
| timeout 5 openssl s_client -tls1 -CApath firefox_ca_certs -quiet -showcerts -connect $1:443 -servername $1 2&gt;cert_chains2/$1 &gt; /dev/null
[/sourcecode]
This is CPU intensive, and overnight it cranked through all the sites.  Example output for www.bankofamerica.com is:
<pre>
depth=3 /C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
verify return:1
depth=2 /C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=(c) 2006 VeriSign, Inc. - For authorized use only/CN=VeriSign Class 3 Public Primary Certification Authority - G5
verify return:1
depth=1 /C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=Terms of use at https://www.verisign.com/rpa (c)06/CN=VeriSign Class 3 Extended Validation SSL CA
verify return:1
depth=0 /1.3.6.1.4.1.311.60.2.1.3=US/1.3.6.1.4.1.311.60.2.1.2=Delaware/2.5.4.15=Private Organization/serialNumber=2927442/C=US/postalCode=60603/ST=Illinois/L=Chicago/streetAddress=135 S La Salle St/O=Bank of America Corporation/OU=Network Infrastructure
verify return:1
read:errno=0
</pre>

Files are available at <a href="https://www.dropbox.com/s/kowtza4kucx12vn/cert_chains.tar.gz">cert_chains.tar.gz (22MB)</a>

I grepped for all files that do not contain the string "verify error" to discover only 245132 sites had SSL which could be verified.  Download at <a href="https://www.dropbox.com/s/5jw6biukka8zvc1/verified_ssl">verified_ssl (4MB)</a>.

<h3>Looking for sites signed by TURKTRUST</h3>
I then grepped through these files for the Turktrust cert (just grepped for "RKTRUST"), and this time came up with <b>only 31 sites</b>!  This list is available at <a href="http://pastebin.com/ZV2gQy22">http://pastebin.com/ZV2gQy22</a>

This list is smaller than the previous 50 I had found, because it turns out that some of the sites I found last time do not have verifiable certs (such as <a href="https://www.mydnn.ir">https://www.mydnn.ir</a>).

All of these sites had the following certificate chain after the leaf node:
<pre>
depth=2 /CN=T\xC3\x9CRKTRUST Elektronik Sertifika Hizmet Sa\xC4\x9Flay\xC4\xB1c\xC4\xB1s\xC4\xB1/C=TR/L=Ankara/O=T\xC3\x9CRKTRUST Bilgi \xC4\xB0leti\xC5\x9Fim ve Bili\xC5\x9Fim G\xC3\xBCvenli\xC4\x9Fi Hizmetleri A.\xC5\x9E. (c) Kas\xC4\xB1m 2005
depth=1 /CN=T\xC3\x9CRKTRUST Elektronik Sunucu Sertifikas\xC4\xB1 Hizmetleri/C=TR/O=T\xC3\x9CRKTRUST Bilgi \xC4\xB0leti\xC5\x9Fim ve Bili\xC5\x9Fim G\xC3\xBCvenli\xC4\x9Fi Hizmetleri A.\xC5\x9E. (c) Kas\xC4\xB1m  2005
</pre>

<h3>Finding all CAs in use by the homepages of the top 1M sites</h3>
It is important to note that both of those certs can technically sign certificates pretending to be any site.  Root CAs and Intermediate CAs have the same power.  The only difference is a root CA is stored in the browsers, whereas the Intermediate CAs need to be signed by a root CA, or in a chain of Intermediate CAs that ultimately gets to the root.

I searched through my data for all of the CA's, and posted it at: <a href="http://pastebin.com/btSeU9Cd">http://pastebin.com/btSeU9Cd</a>.  The counts are the number of times that CA appeared in my data, which unfortunately is going to be slightly higher in some cases than is correct (ex. for TURKTRUST the count is 32 instead of 31 because for some reason www.gununtatili.com shows the same cert chain twice in my data).

<h3>How did Google discover Turktrust's misuse?</h3>
Although not announced anywhere how Google really discovered this, the belief is that Google used its <a href="http://www.imperialviolet.org/2011/05/04/pinning.html">Public key pinning</a>.  Google has some white-listing for it's own certificates in Chrome, and if you have Chrome configured to report errors home, it can inform Google about this.  See the comments on this <a href="https://freedom-to-tinker.com/blog/sjs/turktrust-certificate-authority-errors-demonstrate-the-risk-of-subordinate-certificates/">post</a> for more info.

<h3>Interesting tools</h3>
I also discovered Moxie Marlinspike's interesting idea called <a href="http://convergence.io/index.html">Convergence</a> which is a Firefox add-on.  I highly recommend watching his <a href="https://www.youtube.com/watch?v=Z7Wl2FW2TcA">Blackhat 2011 talk on it</a>.

<h3>Conclusions</h3>
I am of the opinion that the way we assign trust in SSL is broken: We equally trust too many entities which all have the same power.  Default web browsers do not have the ability to warn the user when the certificate chains are weird.  It is only after someone somehow discovers issues with the a CA, that a revocation can be sent out.  Pro-active solutions to find "weirdness" are not common knowledge (like the various browser add-ons).  When CAs make mistakes, or are abused, like this, it can be very costly (like in the case of Diginotar going bankrupt).  It is unclear to me why people have gone after root CAs though instead of just the Intermediate CAs which are more plentiful and likely easier to own?

I do not plan on posting more about CAs.  I'll leave this work up to the pros at the <a href="https://www.eff.org/observatory">SSL Observatory</a> and elsewhere. :)

I need to look at these "crypto protection" tools and techniques more.  I now feel a more pressing need to become more knowledgeable about crypto.  How can I "pentest" crypto?  What are the common implementation flaws?  How can problems be pro-actively identified, such as Google discovering their certs being misused?  How can I add additional layers of protection to my crypto usage, such as the various browser add-ons?  Are we going to have to have "defense-in-depth" solutions for crypto abuses, in the same we have multiple defensive layers against malware and exploits?
