---
layout: post
title: Looking for security trouble spots in Go code
categories: []
tags: []
status: publish
type: post
date: 2015-04-12 14:00:00 -06:00
published: true
---

Different languages have certain areas where mistakes are commonly made, and which code auditors focus on.  With C, you might grep for `strcpy` and `memcpy`.  With ruby, you might look for regex that use [^ and $ instead of \A and \z](http://guides.rubyonrails.org/security.html#regular-expressions).  The use of those functions or idioms are not always vulnerabilities, but are good places to check first.  I decided to look for such trouble spots for Go (golang).  **I did not find any**.  

Go has a lot going for it in terms of security, such as [no pointer arithmetic](https://golang.org/doc/faq#no_pointer_arithmetic), no manual memory management, a standard library that seems fairly well thought out and purposefully excludes things that are misused (ex. [no ECB mode for AES](https://code.google.com/p/go/issues/detail?id=5597)), and put together by a team of seasoned veterans.  However, one must also consider that part of the reason it does not have known trouble spots is perhaps due to it being such a young language that it has not had time for a lot of mistakes to pile up, or for it to be misused in unexpected ways. 

With that, let's look at some issues that have been found.  



**Update 2014.04.15:** *The initial post of this article stated go get defaults to http, but has been corrected to state it falls back to http.*

Fixed issues
============
CVE-2014-7189
-------------
The Go runtime has only one CVE, [CVE-2014-7189](http://www.cvedetails.com/cve/CVE-2014-7189/), which was introduced in Go 1.1 and fixed in Go 1.3.2 on 9/25/2014.  This vuln was relevant to client certificates, which is not a common thing to do, although you can find an example [here](http://www.hydrogen18.com/blog/your-own-pki-tls-golang.html) of using Go with client certs.  The vulnerability is described as "If the server enables TLS client authentication using certificates (this is rare) and explicitly sets SessionTicketsDisabled to true in the tls.Config, then a malicious client can falsely assert ownership of any client certificate it wishes."


RCE on play.golang.org
----------------------
In 2013, Alex Reece of the Plaid Parliament of Pwning (PPP) wrote about [Exploiting a Go Binary](http://ppp.cylab.cmu.edu/wordpress/?p=1087) wherein geohot was able to write Go code that would give an arbitrary write in memory, allowing for Remote Code Execution (RCE).  This is only useful in cases where you can get someone to compile and run Go code of your choosing, given that restriction, this is much like Java client exploits on browsers, but there are no Golang browser plugins, so this could only really be used on play.golang.org or google app engine.  Unless you are receiving and running arbitrary Go code, you don't need to worry much about problems like this.  This appears to have been fixed.


Bug bounties
------------
The only bug bounty I know of for Go code, is [Ethereum's bug bounty](https://www.crowdcurity.com/ethereum) for their client.  The description of Ethereum is a jumble of buzzwords such as "web 3.0" and "turing complete cryptocurrency", but for our discussion we don't need to know what it is.

One bug bounty awarded was for an [out of bounds read](https://bounty.ethdev.com/users/WH8Pa5z8aL3LCs6jE).  These will only cause crashes in Go, so it's a just a DoS, and not that interesting.

More interesting are the [other two bounties](https://bounty.ethdev.com/users/6fqhe7yn6XDCBr6wM).  One was for using a signed int and expecting it be used with values greater than zero.  This allowed for manipulation and abuse of an important value.  Although not used in this case, it is important to point out integer overflows do not cause any exceptions in Go and will wraparound. Most languages do not throw exceptions,  but is possible in C# (there is a project setting to check for this).  Also, the size of an `int` in Go is dependent on the processor, so the following code may have different outputs depending on the system architecture:

```go
var i int = 2147483647
i += 1                
fmt.Println(i)         // Prints -2147483648 or 2147483648
```

The other was an issue related to misuse of crypto wherein a nonce (should be a random number) was generated by xor'ing the private key with the message, thereby leaking information about the private key.  Oddly, the founder of that project actually forked and modified a library in order to implement this questionable [change](https://github.com/obscuren/secp256k1-go/commit/7a58ba75df031897c4cb94bb3b7792959fc06c22#diff-6aebad42932d2e784d6882a9ac024599R126).  The finder of these bugs was awarded 10BTC (about $2300) for each.  Neither of these issues are specific to Go.



Unfixed issues
==============

No ASLR 
-------
Part of the reason the RCE vulnerability mentioned previously was possible is that Go binaries are not compiled with the standard protections such as ASLR, and on Windows it additionally lacks DEP.  This was also discussed by the PPP in a post from 2012, titled [Securing and Exploiting Go Binaries](http://ppp.cylab.cmu.edu/wordpress/?p=667).  It appears for ELF binaries the stack is no longer executable. For demonstration purposes, it was shown that a vulnerable C library, when linked to Go application, does not have the security mitigations normally in place for modern compiled code.  The danger of not having this means that if you have a library that has a memory corruption vulnerability, often these mitigations will make the vulnerability unexploitable, or only probabilistically exploitable.  Although Go itself may not have these memory corruption vulnerabilities, they should still take these precautions since people sometimes may link against non-Go libraries.  

As an example, Don A. Bailey of Lab Mouse Security found such a [vulnerability](http://osdir.com/ml/opensource-software-security/2014-07/msg00142.html) in a real library (the LZ4 compression library) which was interfaced with via a Golang library ([golz4](https://github.com/cloudflare/golz4)) from Cloudflare and [wrote about](http://blog.securitymouse.com/2014/07/bla-bla-lz4-bla-bla-golang-or-whatever.html) his exploitation of it.

We should not expect ASLR to be added any time soon, because in a discussion about this, Russ Cox (who works on the Go runtime) [commented](https://groups.google.com/d/msg/golang-nuts/Jd9tlNc6jUE/6dLasvOs4nIJ):

> "Address space randomization is an OS-level workaround for a 
language-level problem, namely that simple C programs tend to be full 
of exploitable buffer overflows.  Go fixes this at the language level, 
with bounds-checked arrays and slices and no dangling pointers, which 
makes the OS-level workaround much less important.  In return, we 
receive the incredible debuggability of deterministic address space 
layout.  I would not give that up lightly."


go get falls back to http
-------------------------
[Dmitry Chestnykh](https://twitter.com/dchest) opened an [issue](https://github.com/golang/go/issues/9637) about how the `go get` command, which is used to download libaries, will fall back to using http, unless the files are downloaded from github.com, code.google.com, bitbucket.org, launchpad.net, or JazzHub (hub.jazz.net/git).  The threat here is that developers could be MiTM'd so that access to https is blocked, causing the `go get` to fall back to http, allowing for the files to be manipulated. This is not so much a language problem, but something to be aware of.


Conclusion
==========
Unless you are using compiled libraries (derived from C code or other languages that aren't memory safe), you are safe from buffer overflows, use-after-free's, and other memory safety bug classes.  This still leaves the whole [OWASP Top 10](https://www.owasp.org/index.php/OWASP_Top_10).  However, there are not known trouble spots in Go code from a security perspective.

To not leave you completely empty-handed, if you are auditing web server code, some simple things you should check for are:

- CSRF protection via the [nosurf](https://github.com/justinas/nosurf) library.
- XSS mitigation via HTTP headers via the [unrolled/secure](https://github.com/unrolled/secure) library.
- Secure session management via [gorilla/sessions](http://www.gorillatoolkit.org/pkg/sessions) which will result in a call to `sessions.NewCookieStore([]byte("something-very-secret"))`. Make sure something random and secret is used for that byte array, as it will be used in an HMAC (using SHA256) to protect the cookie from tampering.
