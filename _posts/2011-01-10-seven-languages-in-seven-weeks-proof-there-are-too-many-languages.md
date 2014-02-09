---
layout: post
title: ! '"Seven Languages in Seven Weeks": Proof there are too many languages'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1294714303'
  _edit_last: '2'
---
I recently finished reading "<a href="http://www.amazon.com/Seven-Languages-Weeks-Programming-Programmers/dp/193435659X">Seven Languages in Seven Weeks</a>".  If you've ever read "<a href="http://www.amazon.com/Pragmatic-Programmer-Journeyman-Master/dp/020161622X">The Pragmatic Programmer: From Journeyman to Master</a>", then you've heard that software developers should try to learn one new programming language per year.  I decided to pick up the Seven Languages book so I could apply this in fast forward.  The point of learning new languages so frequently isn't so you actually become experts in them.  <b>No one after 10 years of programming, actually uses 10 programming languages</b> on a daily basis.  You probably have one or two core languages that you use, and maybe one language for quick scripting, if you're core language isn't a scripting language, and then you make use of a couple of things which are arguably languages (especially considering how you're using them), such as bash and batch scripting, excel, SQL, various config files, etc.

The Seven Languages book discusses <b>Ruby, Io, Prolog, Scala, Erlang, Clojure, and Haskell</b>.  It doesn't teach you how to get the compiler up and running, or the basic syntax for any of the languages, and it's a good book for that reason.  It focuses on teaching you the things that you would ultimately get out of learning those languages, which is What makes them different? Why were they created? and What are they good for and why?

However, after reading through the book, I've decided <b>I'm happy not learning all those other languages</b>.  Ruby is nice because of it's user base which gives it a lot of support and a bunch of apps to leverage, but I already know perl and python, and Ruby doesn't really seem to offer anything unique. Prolog is sort of cool, but not practical because it's an all or nothing approach.  With Prolog you tell it the constraints you have for a problem, for example a scheduling problem, and it tries to find a solution, and if it can't, then you have absolutely no solution, not even a suboptimal one.  As for the rest of the languages: No interest.

The author tries to make the case frequently that these languages make multi-threaded apps safer and easier to write.  However, any time someone comes along with a new programming language to solve some problem, you have to ask: Why didn't you just write a library for a pre-existing language?  The answer is often "I wanted to make sure the user could never do X anymore."  So great, you made a whole new language, with it's own syntax I have to learn, in order to remove a feature?  That's a waste of time.  It sucks having to write a new language and ensure you have the order of operations working correctly, and write all the string functions, and document the language, etc.  It's a lot of work, and will never be nearly as featureful as the other languages out there.

I've written programming languages with their own interpreters, and I've always had mixed feelings though out the process, and even when I've finished.  You feel smart afterward, but sometimes not productive.  I'm a software developer for production software.  I need code that works, and that I can write fast.   There are only a dozen widely used languages because the value of a network becomes greater the larger it gets (Metcalfe's Law for those academics reading this).  The more people that use a language, the more libraries they write, the more support you can get, and this means more people will use the language.  Similar situation for any OS, application, or anything people use.  Rule #1 for being productive: Use what other people use.

A lot of languages focus on things that are only "cool" for academics.  Example: Recursion.  <b>Recursion is impractical.</b>  The Seven Languages book advocates recursion, as it is discussing languages that derive from Lisp, and Lisp loves recursion.  Recursion wastes stack space.  <b>Wanna know how to make an O(n) time efficient algorithm run in O(1) space?  Make it iterative.</b>  I've never seen a recursive algorithm that was better than the iterative solution.  I get recursion, I can "think" in it, but it's not efficient.  Sure it's "fun" and maybe it looks cool for toy problems, but increase the data set size to something meaningful, and watch things die.

I'll stick to my imperative languages, but thank you to the author, Mr. Tate, for helping me quickly prove to myself that I really am not missing out on anything.  I am an advocate of learning other languages, but make sure there is a reason for doing so, and choose the right languages to learn.
