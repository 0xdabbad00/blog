---
layout: post
title: Endpoint Threat Detection Standards
categories: []
tags: []
status: publish
type: post
published: true
---

The most vocal player in the endpoint threat detection standards game is Mitre.  Unfortunately, they are also the least useful as they don't provide any tools that use the formats they create.  For a long time this meant no one used their formats, but it seems recently the US government (who paid Mitre to make these formats) has started forcing companies to supply data in these formats.  The docs on these formats are lengthy and never give a simple explanation, so this is my simplification.

<h3>Terms</h3>
<ul>
<li><b>Stateful properties</b>: These  are the currently running processes, files that exist, registry keys that exist, etc.  These can be acquired by running periodic scans on a system, or running one scan after an incident.  Think of this as data you obtain from a memory image or hard-drive image.
<li><b>Events</b>: These are acquired by monitoring a system. These are generated as processes are created or terminated, files are created, written to, read from, or deleted, etc.  Think of this as <b>procmon</b> data from the Sysinternals tools.
</ul>

<h3>Formats</h3>
<h4>OpenIOC</h4>
<a href="http://www.openioc.org/">OpenIOC</a> = stateful properties + a threat name.  It is a Mandiant format that is useful for incident response.

<h4>CybOX</h4>
CybOX = stateful properties + event data.  It's not really meant to be used by itself, but rather should be used as part of one of Mitre's other formats.  There are 88 object types possible that range from common things like files and processes, down to oddly specific things like "PDF File" and "Windows hook".  Although mostly focused on Windows, some objects are Unix specific.

<h4>MAEC</h4>
MAEC = CybOX + a threat name.  It is used with some Advanced Threat Protection products, such Cuckoo Sandbox.

<h4>STIX</h4>
STIX = MAEC + whatever you want.  The STIX samples on Mitre's site look like MAEC data plus lot's of meta-data including long html descriptions of pieces of the content.

<h4>TAXII</h4>
This is just how you should send STIX data.  It is a publish/subscribe model built on HTTP/S.  Luckily, Mitre actually made an <a href="https://github.com/TAXIIProject/yeti">example implementation</a> of this.


<h3>Summary</h3>
<table>
<tr><th>Format
    <th>Stateful properties
    <th>Event data
    <th>Malware/threat description
    <th>Kitchen sink
<tr><th>Procmon output
    <td bgcolor="#FF8080">No
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#FF8080">No
    <td bgcolor="#FF8080">No
<tr><th>tasklist output
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#FF8080">No
    <td bgcolor="#FF8080">No
    <td bgcolor="#FF8080">No
<tr><th>CybOX
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#FF8080">No
    <td bgcolor="#FF8080">No
<tr><th>OpenIOC
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#FF8080">No
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#FF8080">No
<tr><th>MAEC
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#FF8080">No
<tr><th>STIX
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#99FF99">Yes
    <td bgcolor="#99FF99">Yes
</table>

<br>