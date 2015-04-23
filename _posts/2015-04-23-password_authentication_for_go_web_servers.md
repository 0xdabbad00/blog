---
layout: post
title: Password Authentication for Go Web Servers
categories: []
tags: []
status: publish
type: post
date: 2015-04-23 14:00:00 -06:00
published: true
---

The stackoverflow question "[How are people managing authentication in Go?](http://stackoverflow.com/questions/25218903/how-are-people-managing-authentication-in-go)" has had a few thousand views.  Go's framework's ([beego](http://beego.me/), [goji](https://goji.io/), [revel](http://revel.github.io/), [martini](http://martini.codegangsta.io/), [negroni](https://github.com/codegangsta/negroni), [gin](http://gin-gonic.github.io/gin/)) do not have anything you should use built-in.  Other languages have a common capability for a "classic" password authentication scheme.

A "classic" scheme requires a username (could be email address) and password, that is stored in a database (with the password hashed).  A cookie is stored in the user's browser to identify them, so they don't need to retype their credentials, and that cookie information is verified on the server.  **This post will look at what is available in Go (golang).**

This will not get into the debate about not using passwords at all (such as using [password reset emails only](http://sakurity.com/blog/2015/04/10/email_password_manager.html), using [client certificates](https://www.scriptjunkie.us/2014/10/replacing-passwords-with-easyauth/), or using things like OAuth).  It's mostly just to help those auditing existing systems.


Checklist
=========
Things to look for in classic web authentication systems for Go.

Server uses TLS
---------------
Not Go specific, but required. Go has capabilities to run as a TLS server, but just put a load balancer and reverse proxy in front (like nginx).

Passwords are hashed, salted, and the hashing is work intensive
---------------------------------------------------------------
This ensures if an attacker gains access to the database, that they cannot log in as other users or compromise those user's accounts on other sites.  The salting (adding random data to a password) and added work is given by using [bcrypt](http://en.wikipedia.org/wiki/Bcrypt), [scrypt](http://en.wikipedia.org/wiki/Scrypt) or [PBKDF2](http://en.wikipedia.org/wiki/PBKDF2) which have both these traits built in and have a "cost" parameter to increase the work.  Those three algorithms are pretty equivalent from an auditing perspective, but if you're building new system, my preference is for **bcrypt**.

Some more recent systems also add "pepper" to their password hashing and salting.  For example, [here](https://github.com/plataformatec/devise/blob/8d48bcd594058049f0976b36764247d6286650af/lib/devise/models/database_authenticatable.rb#L8) is pepper being used in Ruby on Rail's [Devise](http://devise.plataformatec.com.br/) authentication solution. A pepper is similar to a salt in that you are simply concat'ing a string to something and then hashing it, but this is not stored in the database and is constant across all passwords.  The benefit of this is added hassle for attackers that successfully pull off a SQL injection attack.  Another similar solution (in terms of threats it works aganst) is to use a separate database for authentication data so an injection attack on one won't impact another.

The bcrypt library you should use can be found at [golang.org/x/crypto/bcrypt](https://golang.org/x/crypto/bcrypt).  You may see imports that reference it's old location at code.google.com/p/go.crypto/bcrypt.  You'll generate a hash with:

```go
hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
```

The bcrypt library automatically creates a salt.  If you want a pepper too, use:

```go
bcrypt.GenerateFromPassword([]byte(password + "my secret pepper"), bcrypt.DefaultCost) // TODO MUST replace pepper string
```

The output hash will look like:

```
$2a$10$zg0oPq.BwHZb8IRdkBqm2ubhN9b.w5wDYVmKKfKYgvxXY2eumdrJS
```

where the `$10` identifies the cost, and the garble after the third `$` is the base64 encoded salt and hash.  Using this bcrypt library will take care of most of the mistakes you might make. For example, it generates a random salt for you [here](https://github.com/golang/crypto/blob/1fbbd62cfec66bd39d91e97749579579d4d3037e/bcrypt/bcrypt.go#L144).

So include in your database a field for `passwordHash` that is 60 characters. You should not have a `passwordSalt` field.  Ensure that for user registration and logon that the plain-text password supplied by the user is not used anywhere else (hash it and forget it).


Prevent cookie tampering
------------------------
Data about who the user is will be stored in a cookie that is cryptographically "signed" with a secret from the server.  This is done via an HMAC-SHA256 signature.  **Ensure this secret value is truly random and secret**.  

Using [github.com/gorilla/sessions](https://github.com/gorilla/sessions) takes care of this for you as it will use [github.com/gorilla/securecookie](https://github.com/gorilla/securecookie) behind the scenes which adds the HMAC [here](https://github.com/gorilla/securecookie/blob/ab638a3cc27c77beedde96fd004f6c65b7d35211/securecookie.go#L158), where the hash [defaults](https://github.com/gorilla/securecookie/blob/ab638a3cc27c77beedde96fd004f6c65b7d35211/securecookie.go#L51) to a sha256, and the cookie's default age is 30 days.  You need to specify the secret.  The code should have a line like:

```go
cookiejar := sessions.NewCookieStore([]byte(your_secret_key))
```



Cookie should include user secret
---------------------------------
I've seen some authentication systems that only keep usernames in the cookie, and because it is signed, an attacker can't just change the username and login in as a different user.  However, if the attacker discovers (or cracks) the signing secret, then they can, so you should also include some secret that is specific to that user.  For example, Django hashes the user's password hash with a salt (so yes, two salts involved since the password hash involves a salt) and stores that in the cookie ([here](https://github.com/django/django/blob/0f7f5bc9e7a94ab91c2b3db29ef7cf000eff593f/django/contrib/auth/__init__.py#L113) is the cookie storage, and [here](https://github.com/django/django/blob/0f7f5bc9e7a94ab91c2b3db29ef7cf000eff593f/django/contrib/auth/models.py#L256) is the creation of the user secret).  So even if an attacker knows the secret key used for signing cookies, they can't log in as other users. 



All authentication comparisons are time constant
------------------------------------------------
This is to prevent timing attacks. Honestly, I'm unconvinced this is a real threat in most scenarios, but better safe than sorry.  For checking the password hashes that used bcrypt, you should use `bcrypt.CompareHashAndPassword` which uses `subtle.ConstantTimeCompare` behind the scenes.  Use `subtle.ConstantTimeCompare` anywhere else security values are checked (for example, `gorilla/securecookie` uses this to check the HMAC signatures).

For those that like rosetta code, here are some constant time comparisons in different languages:

- Ruby on Rails calls [`secure_compare`](https://github.com/rails/rails/blob/1d9ebec0a9e84aa680313b17ceb800f1b10df3b9/activesupport/lib/active_support/security_utils.rb#L15)
- Python's tornado library calls [`_time_independent_equals`](https://github.com/tornadoweb/tornado/blob/7dc9f25532c2281158caa4c1ade2562b7bb405ce/tornado/web.py#L2968)
- In Python 3.3, it calls C code via `hmac.compare_digest` to [`_tscmp`](https://github.com/python/cpython/blob/c71e8b81f1f4d349d1a24a6fe162cbbecedff8f0/Modules/_operator.c#L176)

Because I go full tin-foil hat some times, I checked the generated x86-64 code for `subtle.ConstantTimeCompare`, since I was worried about compiler optimizations.  This is not an issue as Go doesn't do these types of optimizations, but in languages like C you would want a pragma to disable optimizations or use the `volatile` keyword as that is what `_tscmp` uses.  It's not well-known, but Go actually does have some pragma's such as `go:nosplit`, `go:noescape`, `go:nowritebarrier`, `go:linkname` and some others.  These aren't really documented so you shouldn't use them.  An example of their use in the run-time is [here](https://github.com/golang/go/blob/9402e49450d57eb608f03980e7541602a346e5ae/src/runtime/os1_windows.go#L176) and you can see the lexer parsing for them [here](https://github.com/golang/go/blob/88c08b06b97296e41fc3069f4afbc86d24707b05/src/cmd/internal/gc/lex.go#L1576).

As long as I'm admitting my trust issues to the world, you may also be interested that Go gets random correctly as you can see [here](https://github.com/golang/go/blob/c007ce824d9a4fccb148f9204e04c23ed2984b71/src/crypto/rand/rand_windows.go#L26) for Windows where it calls the OS API's `CryptAcquireContext` and `CryptGenRandom`.  It pulls from [`/dev/urandom`](https://github.com/golang/go/blob/c007ce824d9a4fccb148f9204e04c23ed2984b71/src/crypto/rand/rand_unix.go#L23) on other OS's, except Plan 9.  I don't know much about Plan 9, but if you're a cryppie and know that OS, take a [peak](https://github.com/golang/go/blob/c007ce824d9a4fccb148f9204e04c23ed2984b71/src/crypto/rand/rand_unix.go#L94).




Other features
--------------
- **Brute force protection**: Although bcrypt has a cost function to make it slower, an attacker could just multi-thread his brute-forcing code that tries to logon to your server with multiple credentials at once.  So you'll want to do some sort of throttling, or add time-outs for IP's with too many failures, or add a CAPTCHA.  You want to avoid locking out legitimate users though.

- **Password reset emails**: When a user forgets their password, a URL with a special token will be emailed to them. Ensure this token is disabled after it is first used, and/or after a few hours.  Apply all the same precautions to this as you would other security information.

- **Two-factor authentication**: This will also help with any brute-forcing. If an attacker get's access to your database though, then it's no longer useful, as you will have likely have stored your 2FA seed in your DB.  That assumes you're using something like Google authenticator as opposed to SMS messages.  If you use SMS, make sure an attacker doesn't burn your bank account by causing a billion SMS's to be sent. 


Current Go Options
==================

HTTP Basic Auth
---------------
To start, let me correct what I said earlier: Technically, [beego](http://beego.me/) does have some authentication functionality in [github.com/astaxie/beego/plugins/auth/basic.go](https://github.com/astaxie/beego/blob/master/plugins/auth/basic.go), but you shouldn't use it for a real site.  One reason why is it requires your application to have code like

```go
beego.InsertFilter("*", beego.BeforeRouter,auth.Basic("username","secretpassword"))
```

The problem here is that you are required to include the plain-text password in your application, and **web servers should never store plain-text passwords**.  Likewise, there is http middleware that can be used with [goji](https://goji.io/) and others at [github.com/goji/httpauth](https://github.com/goji/httpauth) which again requires the passwords to be stored in plain-text.  These are just Basic HTTP Authentication as defined in the original HTTP [RFC 1945](http://tools.ietf.org/html/rfc1945#section-11), without the use of cookies or session management.


Other options
-------------
There is a project [github.com/apexskier/httpauth/](https://github.com/apexskier/httpauth/) which get's a lot things right if you want to look at it for how some of the functions discussed above are used, but it's only a starting point for your web authentication needs, and it needs to store a user secret in the cookie along with just the username.

A more recent, and much more extensive, project is [authboss](https://github.com/go-authboss/authboss), but I have not looked at it too closely and it may be doing too much for your needs.