---
layout: post
title: Go Everyday
categories: []
tags: []
status: publish
type: post
date: 2014-12-27 14:00:00 -06:00
published: true
---

I've been coding in Go everyday for the past 3 months, for [Summit Route](https://summitroute.com). This post describes how I use Go and what works for me, some of which differs from what I've seen in a lot of other tutorials.  Tutorials give you the quick solution that might not be the best when you use it every day.  This might help some others and the experts might give me some better ideas.

- [Tip 1: Use the Go Package Manager for Revision Locking](#tip1)
- [Tip 2: Use direnv for per project GOPATHs](#tip2)
- [Tip 3: Use the Atom editor with the go-plus plugin](#tip3)
- [Tip 4: Use the latest Go](#tip4)
- [Tip 5: Debug with gdb](#tip5)


<a name="tip1"></a>
Tip 1: Use the Go Package Manager for Revision Locking
======================================================
To download the packages your project uses the common advice is to use `go get`.  This will scan your source files and download any imports you use.  Unfortunately, it pulls the HEAD from the repo.

When you're coding for production servers, this creates problems when you want to always use a specific branch or tag of a package.  One reason to avoid HEAD is you may want to ensure that the only changes to your service are the ones you make, so when a problem appears you don't realize that one of your servers is running a slightly different library than the others.

The solution I use for this is the **Go Package Manager** ([gpm](https://github.com/pote/gpm)).

Go Package Manager
------------------
[gpm](https://github.com/pote/gpm) is not like NPM, or pip, or many other package managers that pull from a centralized package manager site.  Insted gpm is a simple bash script to allow you to specify specific library versions (commits, tags) you want to use and it just pulls from github.com or whichever server you've specified.   You can read more [here](http://technosophos.com/2014/05/29/why-gpm-is-the-right-go-package-manager.html).  It allows you to specify the projects you want to use and their specific tag or commit.  As an example, here is part of a `Godeps` file.

```
github.com/golang/glog d1c4472bf2efd3826f2b5bdcc02d8416798d678c
github.com/zenazn/goji v0.8.1
```


<a name="tip2"></a>
Tip 2: Use direnv for per project GOPATHs
=========================================
If you're using the Go Package Manager, you may want to set your GOPATH variable for each project. Many people use a single GOPATH for multiple projects, so this may not be needed for your setup.

The GPM developer wrote a tool called [GVP](https://github.com/pote/gvp) (Go Versioning Packager) to assist with this.  It's simply a bash script that sets your GOPATH.  You `cd` to your project, then run `gvp`, and your `GOPATH` is set.  I find it easier to use [direnv](http://direnv.net/) which will automatically do this for me whenever I `cd` into the directory.

My `.envrc` file is

```
export GVP_DIR="$(pwd)/.godeps"
export GOBIN="$GVP_DIR/bin"
export GOPATH="$GVP_DIR:$PWD:`echo $PWD | sed 's/src.*//'`"
export PATH="$GOBIN:$PATH"
```

My `GOPATH` is set to the current directory and any parent `src` directory. So when I run

    cd /gocode/src/SummitRoute/WebServer

 my `GOPATH` is set to

    /gocode/src/SummitRoute/WebServer/.godeps:/gocode/src/SummitRoute/WebServer:/gocode

This allows for the following (simplified) directory structure.

- /gocode/src/SummitRoute
    - WebServer
        - server.go
    - CallbackServer
        - server.go
    - Database
        - model.go

Now in both my `server.go` files I can share `SummitRoute/Database`:

```
import "SummitRoute/Database"
```





<a name="tip3"></a>
Tip 3: Use the Atom editor with the go-plus plugin
==================================================
All the value of most editors is in in it's plugins and for Atom you'll want to install [go-plus](https://github.com/joefitzgerald/go-plus).  It provides syntax high-lighting and will autoformat your code on save, so it looks standardized.  go-plus will also lint your code so it checks for lot's of bugs you might have.  You can also get plugins for code completion.

There are lots of other editor options, with many providing at least syntax high-lighting.  There are also some sliglty more featureful options, such as the[Zeus](http://www.zeusedit.com/go.html) IDE but it is Windows only.  The additional capabilities it adds over Atom are code following (go to declaration) and some debugging support via integration with GDB.



<a name="tip4"></a>
Tip 4: Use the latest Go
========================
Chances are if you're on Linux, your package manager installed an older version of Go.  You should use the latest version for performance, security, and compatibility reasons.  [Go 1.4](https://blog.golang.org/go1.4) was released on Dec 10, 2014.

Download and extract the latest version from [https://golang.org/dl/](https://golang.org/dl/)

Ensure you have the latest:

```
$ go version
go version go1.4 linux/amd64
```


<a name="tip5"></a>
Tip 5: Debug with gdb
=====================
I love print statements as much as anyone for debugging, but sometimes you need a debugger.  In Go, that means using gdb.  One of the main reasons to switch to Go 1.4 is the improved debugging experience.
Even in Go 1.3, I'd get issues with gdb not knowing the names of variables and other issues that made gdb not very useful for Go debugging.  With Go 1.4, it's been working much better.

To debug, you'll want to build binaries (do not use `go run server.go` and attach to it).  You'll build and run as follows:

```
go build -gcflags "-N -l" server.go
gdb --args ./server --port=8000
```

That will disable optimizations for the build and then run your code with whatever arguments you might normally use.

When using the GDB debugger for your Go code, you should see be able to set a breakpoint such as on `main.main` (`main` package, `main` function), see your source code, and print the values of your variables.  If you can't, something is broken.
