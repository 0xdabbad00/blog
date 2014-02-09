---
layout: post
title: ! 'Idea: Tool to cluster executables'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1284512895'
  _edit_last: '2'
---
HBGary's <a href="https://www.hbgary.com/community/free-tools/#fingerprint">Fingerprint</a> tool gathers various information from executables for identifying developer trade-craft (I'm working on another post in which I'll review this tool and the rest of HBGary's tools, and discuss them in greater length).  It has an ability to compare files, but it's output isn't very user-friendly (just text).  HBGary's presentations have shown their ability to use this data to generate pretty cluster graphs of malware samples, but there isn't an easy way for the rest of us to.  The idea here would be to just write up how to grep out the relevant data and input to a tool that shows clustering, or possibly slap a nice GUI on top of FingerPrint.  I like command-lines, but there is also a need for GUI's sometimes.  

It'd be REALLY cool if this was a web app, because it would already have the back-end database of files to compare it to.  So you'd see a big map of clusters, with your sample that you uploaded high-lighted, along with some other samples identified to give you some understanding of where your sample was clustered near.  For example, cluster for "Conficker" and "Stuxnet" and even goodware maybe like "7-zip" and "Microsoft Word".  You'd see your sample was near the Conficker cluster and as you zoomed in, those clusters would show names like "Conficker A", "Conficker B", etc.  and you'd see your sample matched "Conficker C".  This might be too much data, and the FingerPrint tool might need to mature to handle packing and other simple tricks better, but it would still be pretty sweet!

<b>Update 2010.09.13</b>: HBGary put their <a href="https://www.hbgary.com/uncategorized/hbgarys-greg-hoglund-speaking-at-black-hat-2010/">presentation from Blackhat</a> online.  Check out the image at 56:30 for an example of a cluster diagram created using data from Fingerprint.
