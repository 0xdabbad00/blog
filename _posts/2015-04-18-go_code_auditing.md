---
layout: post
title: Go code auditing
categories: []
tags: []
status: publish
type: post
date: 2015-04-18 14:00:00 -06:00
published: true
---

In the book "[The Art of Software Security Assessment: Identifying and Preventing Software Vulnerabilities](http://www.amazon.com/Art-Software-Security-Assessment-Vulnerabilities/dp/0321444426)" from 2007, the authors [Mark Dowd](https://twitter.com/mdowd), John McDonald, and [Justin Schuh](https://twitter.com/justinschuh), discuss where to look for common trouble spots that affect all web platforms.  They discuss CGI (when a web server used to directly call an executable), Perl, PHP, Java, ASP, and ASP.NET. They focus on a couple of fairly common application features, where when things go wrong, they usually go horribly wrong, so you should look for these techniques specifically.  This post hopes apply that same methodology to Go (golang).  This post builds on my previous post [Looking for security trouble spots in Go code](http://0xdabbad00.com/2015/04/12/looking_for_security_trouble_spots_in_go_code/) which was focused on looking for problems that would be specific to Go.



SQL Injection
-------------
Go can use parameterized queries which help avoid the problems of SQL injection.  For example, the first line below is a parameterized query which ensures the variable will not be treated as arbitrary SQL, whereas the second is a string concatenation which could cause problems.

```go
db.Query("SELECT name FROM users WHERE age=?", userinput)  // OK
db.Query("SELECT name FROM users WHERE age=" + userinput)  // BAD
```


File Access
-----------
Files can be opened for reading in Go using `ioutil.ReadFile(name)` or `os.Open(name)`.  Path traversals are as usual possible, so you'll need to check if `name` can be set to something like `/../../etc/passwd`.

One problem that affects some languages are null bytes in strings, such that the language allows it, but the underlying OS does not respect it, so you can end up with problems of Poison NULL bytes, as discussed in [Phrack 55](http://phrack.org/issues/55/7.html) by [Rain Forest Puppy](https://twitter.com/j4istal).  In that example he shows the Perl code:

```perl
# parse $userinput
$database="$userinput.db";
open(FILE "<$database");
```

This code set the `$database` variable to the concatentation of the variable `$userinput` with the string `".db"` and opens it.  Normally, this app would want `$userinput` to be set to `"rfp"` so it opens `"rfp.db"`, but if an attacker can set the variable to `"secretfile\0"`, then instead of failing to find `"secretfile\0.db"` it would open and read `"secretfile"`.  

This might sound like an odd thing to be concerned about, but as a somewhat recent example, a quick search showed this affecting JBoss Web Server (a Java app) in 2013, via [CVE-2013-2186](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-2186), and as proof, here is a [patch](https://git.centos.org/patch/rpms!thermostat1-apache-commons-fileupload/d140978788ad80704e0fac41c86a926191e0b05b/SOURCES!CVE-2013-2186-commons-fileupload.patch) that is adding a check for null bytes.

In Go, null bytes are specifically not allowed when opening files, due the code at https://github.com/golang/go/blob/e7173dfdfd1c74b1d301da9b6f295ef99b9cc11c/src/syscall/syscall.go#L45

This is called for any code that passes through a syscall to open a file.  It will result in the error `invalid argument`. 

Some languages, such as PHP, also allow opening of URL's via the file open functions, such as `http://` or `file://` which requires additional consideration.  This is not possible with Go. 


Shell Invocation
----------------
You can run shell commands from Go using `exec.Command(name string, arg ...string)` from the `os` package. As you can expect based on the definition of the function, it is not possible to pass a string such as `echo hello | cat /etc/passwd` to the initial command argument, or any of the parameters.  You are forced to use this function in something like `cmd := exec.Command("echo", "hello", "world")`

Here is a quick exmaple of how this protects you:

```go
userinput := "hello | cat /etc/passwd"
out, err := exec.Command("echo " + userinput).Output() // DUMB, will cause errors
if err != nil {
	fmt.Printf("ERROR: %v", err)
}
fmt.Println(string(out))

// Prints: ERROR: fork/exec echo hello | cat /etc/passwd: no such file or directory
```



File Inclusion
--------------
Scripting languages let you pull in libraries at runtime.  The danger here is if the user has control over the name and thereby cause a malicious script file to be executed.  Go does not allow new packages to pulled in at run-time.  For example, this is not possible:

```Go
if err {
  import "fmt" // BAD: Syntax error
  fmt.Println("Error:" + err)
}
```

This would not work for a lot of reasons, but the big is one is that Go is a compiled language and pulls all it's package into the binary that is created during `go build` or `go run`.  You can however, import DLL's or shared libraries in Go using:

```Go
kernel32, _        = syscall.LoadLibrary("kernel32.dll")
```

The usual concerns apply here about DLL hijacking or if the user for some horrible reason has control over the `LoadLibrary` argument and passes a remote network share address such as `\\evil.com\evil.dll`.




Inline Evaluation
-----------------
Many scripting languages allow you to run `eval()` to execute arbitrary code.  Again, because Go is a compiled language, and not interpretted, this does not affect it.  This is also why Go does not come with a REPL (Read-Eval-Print-Loop) utility, so when you want to test simple piece of code, you always have to write a file and `go run` it.  People have written REPL's for Go, such as [gore](https://github.com/sriram-srinivasan/gore), but these are simply writing temporary files and running `go run` on it behind the scenes (as shown [here](https://github.com/sriram-srinivasan/gore) from gore). 


Cross-Site Scripting
--------------------
Go comes with the package [html/template](http://golang.org/pkg/html/template/) that comes with some functions such as [`HTMLEscapeString`](http://golang.org/pkg/html/template/#HTMLEscapeString) to escape strings so they can't cause XSS issues.  For example:

```go
userinput := "<script>alert(1);</script>"
fmt.Println(template.HTMLEscapeString(userinput))
// Prints: &lt;script&gt;alert(1);&lt;/script&gt;
```

Additionally, you can add the HTTP middleware [github.com/unrolled/secure](https://github.com/unrolled/secure) to add HTTP headers that cause browsers to deny script tags from existing in the content they receive, along with some other checks.


Configuration
------------- 
Languages such as PHP can end up enabling or disabling security settings via configuration files.  Go has a very limited number of environmental variables that it reads from when it runs (GODEBUG, GOMAXPROCS, and GOTRACEBACK).  None of these have security concerns.  Security concerns based on configuration will be application specific, if any config settings exist at all. 


Conclusion
==========
I hope this helps direct your audits of Go code and teaches a little about the language.