---
layout: post
title: ! 'Idea: Tool to rate use of defense-in-depth'
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1284332776'
  _edit_last: '2'
---
I read the follow-up reports of the new Adobe Reader 0-day at <a href="http://www.prevx.com/blog/156/New-Adobe-day-exploit-in-the-wild.html">Prevx</a> and <a href="http://www.vupen.com/blog/">VUPEN</a> (I am unsure how to link to VUPEN's specific articles as opposed to their blog's main site, so try searching for the title "Criminals Are Getting Smarter: Analysis of the Adobe Acrobat / Reader 0-Day Exploit" if the page changes).  These get a little more into the details of what is being exploited, and specifically they note that the exploit is made possible because Adobe is using a DLL that hasn't been compiled to use DEP and ASLR, and also uses the old strcat function instead of the more secure functions like strcat_s and strncat (yeah, you can mess those up too, but at least you're to consider the possible vulnerability).  So first, yeah, this 0-day would be blocked by EMET and isn't using Didier's vuln as I was previously confused about in <a href="http://0xdabbad00.com/2010/09/09/more-malware-signed-by-real-certs/">my post</a> about it. 

Anyway, I think it would be cool to write a tool that could scan an application and give it a score of it's likely security.  Let me explain what it would check:
<ul>
<li>Is DEP and ASLR turned on?  If not, subtract 50 from the score for each instance.  Check for other compiler options that yield safer code.
<li>Is it importing deprecated or older functions?  If not, subtract 50 and subtract 5 for each function or each call to the function.
<li>What compiler was used to compile it?  Visual Studio has had the following major releases 2010, 2008, 2005, 2003, and Visual Studio 6.0 (which came out in 1998).  So subtract the a value depending on how old the compiler is.  The idea here is that the compiler's offer more security features and give better warnings as time progresses.  You may also want to consider the deprecated functions here, because if they're using a modern compiler AND they're using deprecated functions, then they are purposefully flouting the compiler's warnings!
<li>Possibly try to identify code blocks associated with older crypto schemes.  Chances are you need backwards compatibility, but you can probably make that case that anywhere DES is being used, AES should be used instead, so if you see DES usage, but not AES anywhere, then the code probably needs updating.
<li>Is the code signed?
<li>Is the code using Unicode functions or ASCII?  The idea here being that a more mature code base is going to support Unicode.
</ul>

I am not trying to attempt the impossible of being able to tell if an application has vulnerabilities or not, but I think one could write a tool to see how mature and how interested in security a product is.  This was just a quick brain storm, so there are probably lots of other things that could be done programmatically, and lots more could be done with a little human help.  For example, you could make a database of the compilation dates for commonly used DLL's, where the date is the last instance of that DLL that had a specific vulnerability.  Then based on file name or something more intelligent maybe, you could find the commonly used DLL's installed with an application, and determine if they had known vulnerabilities, which would imply they should be updated.

This tool would allow a basic user (or a company's management) to determine how seriously the company takes security (management may be unaware that it's developers aren't focused on this), and if the score is bad, it might motivate the company.  Or for the developer's it would be a good check as they could judge third-party DLLs more easily.  If the tool became popular hopefully companies would either improve, or would respond with an explanation for why their tool scored poorly, which could be used to improve the tool (the tool might be making bad assumptions), or the tool could try to identify the application it was rating and provide the user with a link back to a page with content provided by the company that would explain why their score is bad.

HBGary's <a href="https://www.hbgary.com/community/free-tools/#fingerprint">Fingerprint</a> would be useful to gather input for this tool.
