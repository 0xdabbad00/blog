---
layout: post
title: Choosing Libraries for Go Web Servers
categories: []
tags: []
status: publish
type: post
date: 2015-04-03 14:00:00 -06:00
published: true
---

One of the hardest things about coming to a new language is figuring out which libraries you should use, especially for young languages like Go, where there are a bunch of competing options with no clear winners.  As an example, for Node, there was an [issue](https://github.com/npm/npm/issues/7048) opened for it's npm package manager because there were 127 different bcrypt libraries.  

This post will show some of the different choices I made in the libraries I use for the back-end to my end-point protection product at [Summit Route](https://summitroute.com/).  As quick summary, I use the [goji](http://goji.io/) web framework, some middleware, and the database library [gorp](https://github.com/go-gorp/gorp) (with Postgres for my database).



Web framework: goji
===================
<img width=100px src="/wp-content/goji.png"><br>
Go has a handful of web libraries and frameworks.  The most light-weight option is to nakedly use the built-in [net/http](http://golang.org/pkg/net/http/).  Under the hood, all the other options use this.  

To get the ability to easily include middleware, I chose to go with [goji](http://goji.io/).  I recommend goji for those comfortable with Python's [tornado](http://www.tornadoweb.org/en/stable/) or [twisted](https://twistedmatrix.com/trac/) libraries.

If you're more comfortable with Python's [django](https://www.djangoproject.com/) or Ruby's [Rails](http://rubyonrails.org/), you might be more comfortable with one of Go's heavier web frameworks.  Options here include [revel](http://revel.github.io/), [gin](https://github.com/gin-gonic/gin), and [beego](http://beego.me/).  Beego has heavy usage in the Chinese market among some top companies there.

If you search online for Go web frameworks you'll see references to [martini](https://github.com/go-martini/martini).  The creator of martini decided to mostly stop supporting that library, and created [negroni]([negroni](https://github.com/codegangsta/negroni)). He explained his thoughts on [his site](http://codegangsta.io/blog/2014/05/19/my-thoughts-on-martini/). So you shouldn't use martini.  negroni requires middleware for graceful shutdown and routing capability, so you can decide how much you want built-in versus being more easily extensible.  For me, goji is a good balance.

Looking online, you'll also see references to the [gorilla](http://www.gorillatoolkit.org/) toolkit which adds some additional functionality, and likely you'll pull from this toolkit for it's session library no matter which of the web frameworks you use.

As meaningless as lines of code are, to give you some idea of how much heavier beego is than the other options, using the [`cloc`](http://cloc.sourceforge.net/) command on the git repos for the various projects give the following stats:

<table>
	<tr><th>Project</th><th>Lines of Go code</th></tr>
	<tr><td><a href="https://github.com/astaxie/beego/">beego</a></td><td>26,510</td></tr>
	<tr><td><a href="https://github.com/revel/revel">revel</a></td><td>7,278</td></tr>
	<tr><td><a href="https://github.com/zenazn/goji">goji</a></td><td>4,131</td></tr>
	<tr><td><a href="https://github.com/gorilla/mux">gorilla/mux</a></td><td>2,259</td></tr>
	<tr><td><a href="https://github.com/gin-gonic/gin">gin</a></td><td>2,172</td></tr>
	<tr><td><a href="https://github.com/go-martini/martini">martini</a></td><td>1,685</td></tr>
	<tr><td><a href="https://github.com/codegangsta/negroni">negroni</a></td><td>590</td></tr>
</table>

Part of that extra code in beego is the ORM for database access that it includes.


Goji middleware
===============
Goji doesn't need any extras but using the following middleware provides nice-to-have's very easily.

Goji + secure
-------------
[secure](https://github.com/unrolled/secure) is an HTTP middleware for Go that adds HTTP headers to stop XSS, downgrade attacks, mime type issues, and inline frames.  Your reverse proxy or load balancer may provide these for you if you configured it properly. Amazon's ELB does not, so you'll want to use this middleware for that architecture.  To check your server is providing the correct HTTP headers, test it against [https://securityheaders.io/](https://securityheaders.io/).  This middleware also provides some SSL specific headers, but I recommend using a reverse proxy, such as nginx or Amazon's ELB (Elastic Load Balancer), in front of your service to take care of that and pass you plain HTTP.


Goji + nosurf
-------------
[nosurf](https://github.com/justinas/nosurf) is CSRF protection middleware for Go.  This means your HTML forms will include `csrf_token` values like this:

```html
{% raw %}
<form action="/" method="POST">
<input type="text" name="name">
<input type="hidden" name="csrf_token" value="{{ .token }}">
<input type="submit" value="Send">
</form>
{% endraw %}
```

Goji + glogrus
--------------
[glogrus](https://github.com/goji/glogrus) provides structured logging via [logrus](https://github.com/Sirupsen/logrus) for Goji.  What this means is your logs can be formatted as json so that services like [Loggly](https://www.loggly.com/) or a privately hosted ELK (ElasticSearch + Logstache + Kibana) can parse them more easily without you needing to define the structure of the logs.  This means your logs might look like:

```json
{"level":"info","msg":"Authentication success","user":123,"time":"2014-04-02 19:57:38.562543128 -0400 EDT"}
```


Putting it all together
=======================
Once you decide to use goji, you can just piece things together, but you'll probably want to see some "best practices" for a skeleton on how to set things up and structure where to put your files.  One example is at [github.com/elcct/defaultproject](https://github.com/elcct/defaultproject)  It uses goji, but uses MongoDB, and made some other library choices.  A fork of that, [haruyama/golang-goji-sample](https://github.com/haruyama/golang-goji-sample), uses MySQL with gorp, which is closer to our needs, but still not quite right.

For example, both these projects use the popular [github.com/golang/glog](https://github.com/golang/glog) from Rob Pike (one of the developers of Go), which provides leveled logging as an improvement over the basic logging of the built-in log library.  Levelled logging simply means you can emit info, warnings, and errors, and do some basic filtering on what you'll actually output.  This library is too minimal.  For example, all configuration options must be passed via command-line flags to your application, as opposed to passing configs through a file, which I like to do.  Additionally, I think structured logging is easier to sift through when your logging needs get bigger.  However, when you're just debugging locally, structured logs are harder to read, so I actually use text logs when debugging and structured logs when in production. logrus makes this easy.

Using the [server.go](https://github.com/elcct/defaultproject/blob/master/server.go) from that skeleton project, you can see an **improved version at this gist**:

- https://gist.github.com/0xdabbad00/98bb562f3abbe038cec6


In order to use [github.com/justinas/nosurf](https://github.com/justinas/nosurf), in your `system\middleware.go`, **add the function from this gist**:

- https://gist.github.com/0xdabbad00/1c9c6c293e57d5a24431


Database access: gorp
=====================
Go has basic database access built-in via [database/sql](http://golang.org/pkg/database/sql/) and there are libraries to work easily with popular databases such as:

- Postgres: [github.com/lib/pq](https://github.com/lib/pq)
- MySQL: [github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)
- SQLite: [github.com/mattn/go-sqlite3](https://github.com/mattn/go-sqlite3)

However, you'll likely want to add another layer of functionality on top. I use [gorp](https://github.com/go-gorp/gorp), which is not quite what you'd see in a full ORM.  To show the different libraries, let's assume we have a `users` table and a simple variable `user` defined as:

```go
type User struct {
	ID        int64
	FirstName string
	LastName  string
	Email     string
}
var user User
```

Now I'll show how you'd do the same query in different libraries so you can choose which one's syntax you like:

[database/sql](https://golang.org/pkg/database/sql/)
------------
```go
db.QueryRow("SELECT * FROM users WHERE id = ?", 1).Scan(
	&user.ID,
	&user.FirstName,
	&user.LastName,
	&user.Email)
```

[gorp](https://github.com/go-gorp/gorp)
----
```go
db.SelectOne(&user, "SELECT * FROM users WHERE id = ?", 1)
```
gorp still makes you write SQL, but save's you a bunch of typing.  I don't like abstracting myself away too much from the SQL like you do with the ORM's below.


[beedb](https://github.com/astaxie/beedb)
-----
```go
db.Where("id=?", 1).Find(&user)
```
beedb has been deprecated in favor of beego.orm

[beego.orm](https://github.com/astaxie/beego/tree/master/orm)
---------
```go
user = db.QueryTable("users").Filter("id", 1)
```
beego.orm is used by the [beego](https://github.com/astaxie/beego) web framework.

[hood](https://github.com/eaigner/hood)
----
```go
db.Where("id", "=", "1").Limit(1).Find(&user)
```

[upper.io](https://github.com/upper/db)
--------
```go
db.Find(db.Cond{"id": 1}).One(&user)
```

[squirrel](https://github.com/lann/squirrel)
-------
If you did want to build more complex queries with gorp, you could use squirrel to generate your SQL.

```go
sql, args, _ := sq.Select("*").From("users").Where(sq.Eq{"id", 1}).ToSQL()
db.SelectOne(&user, sql, args)
```

Conclusion
==========
Hopefully this helps those of you new to Go!