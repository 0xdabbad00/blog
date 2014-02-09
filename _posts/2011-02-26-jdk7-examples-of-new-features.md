---
layout: post
title: JDK7 examples of new features
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1325653254'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
Java 7 was released to developers this week (Java 6 was released in Dec 2006).  From what I've read, it's not too exciting.  It has some new features like <b>TLS 1.2 support</b> for more secure network transactions, and some features to save typing for example:

<h3>Diamond collections don't need to repeat types</h3>
[sourcecode language="java"]
// JDK6
List&lt;String&gt; l = new LinkedList&lt;String&gt;();
// JDK7
List&lt;String&gt; l = new LinkedList&lt;&gt;();
[/sourcecode]

<h3>try's can be told what resources to clean-up without a finally</h3>
[sourcecode language="java"]
// JDK6
BufferedReader br = new BufferedReader(new FileReader(path));
try {
  return br.readLine();
} finally {
  br.close();
}

// JDK7
try (BufferedReader br = new BufferedReader(new FileReader(path)) {
 return br.readLine();
}
[/sourcecode]

<h3>Collection literals</h3>
[sourcecode language="java"]
// JDK6
List&lt;Integer&gt; piDigits = Collections.unmodifiableList(Arrays.asList(3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5, 9 ));
// JDK7
List&lt;Integer&gt; piDigits = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5, 9};
[/sourcecode]

<h3>Binary support... sort of</h3>
It'd be awesome if Java would actually support byte's as primitives and unsigned values.  My favorite rant on this is from the <a href="http://www.ragestorm.net/blogs/?p=282">Insanely Low Level blog's post</a>.
[sourcecode language="java"]
// JDK6 (this or similar awkwardness)
int binary = (1&lt;&lt;3) | (0 &lt;&lt; 2) | (0 &lt;&lt; 1) | (1 &lt;&lt; 0);
// JDK7
int binary = 0b1001;
[/sourcecode]

<h3>Underscores in numbers</h3>
Not very exciting at all, but weird when you encounter it.
[sourcecode language="java"]
// JDK6
int ssn = 123456789;
// JDK7
int ssn = 123_45_6789; // underscores get ignored
[/sourcecode] 
