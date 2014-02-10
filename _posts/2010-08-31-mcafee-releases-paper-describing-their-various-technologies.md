---
layout: post
title: McAfee releases paper describing their various technologies
categories: []
tags: []
status: publish
type: post
published: true
---
<a href="http://www.mcafee.com/us/local_content/reports/mcafee_engine_technologies.pdf">http://www.mcafee.com/us/local\_content/reports/mcafee\_engine\_technologies.pdf</a>

It's not very in-depth but it hints at A LOT of things.  It's like a summary of the book "The Art of Computer Virus Research and Defense" but with updates for modern techniques and not all the old 1980s history.   For example, on page 13 it discusses their use of statistical classification and fuzzy fingerprinting.

Also of interest is their reference at the bottom to a presentation titled "Using Markov Models to Detect Code Execution Exploits"  which I found at: <a href="http://papers.filegazebo.com/CARO09_AlmeElser_MarkovDetectExploits.pdf">http://papers.filegazebo.com/CARO09\_AlmeElser\_MarkovDetectExploits.pdf</a>

That presentation basically describes how you can use HMM's (Hidden Markov Models) to determine what is likely to be shell-code, then you take data files and recursively try to disassemble them and then run any disasssembly you get through the HMM to predict if it is shell-code.
