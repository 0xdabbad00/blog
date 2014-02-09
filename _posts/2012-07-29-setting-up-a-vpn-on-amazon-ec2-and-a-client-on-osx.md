---
layout: post
title: Setting up a VPN on Amazon EC2 and a client on OSX
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1375035726'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
Before defcon, I decided to set up a VPN for my internet traffic.  I could not find any good VPN offerings for my needs, as they are all geared towards people doing sketchy things and not wanting to be tracked, or people overseas wanting access to sites that are blocked in their current locations. Most charge monthly fees and most are sketchy.

Amazon EC2 gives you one year free to host a system in their cloud.  I run my server off the default Amazon Linux AMI (Amazon Machine Image) which is based on Redhat.  I mostly followed the directions at: <a href="http://holgr.com/blog/2009/06/setting-up-openvpn-on-amazons-ec2/">Setting up OpenVPN on Amazon’s EC2</a>.

<h3>Amazon EC2 OpenVPN Configuration</h3>
<ol>
<li>Run the following on the server:
[sourcecode gutter="false"]
yum install openvpn
openvpn —genkey —secret /etc/openvpn/openvpn-key.txt
[/sourcecode]

The key file openvpn-key.txt will be copied to the client later.

<li>Create a file <tt>/etc/openvpn/openvpn.conf</tt> with contents:
[sourcecode gutter="false"]
port 1194
proto udp
dev tun
secret openvpn-key.txt

# The server's virtual endpoints
ifconfig 10.8.0.1 10.8.0.2
push &quot;redirect-gateway def1&quot;

keepalive 10 120
comp-lzo
persist-key
persist-tun
status server-tcp.log
verb 3
[/sourcecode]

<li>Run the vpn:
[sourcecode gutter="false"]
service openvpn restart
chkconfig openvpn on
[/sourcecode]

<li>Open up the firewall

<li>Get iptables to route your traffic:
[sourcecode gutter="false"]
iptables -A INPUT -i tun+ -j ACCEPT
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
[/sourcecode]

and change the Security Group in the Amazon EC2 Management Console to open up UDP port 1194:
<a href="http://0xdabbad00.com/wp-content/uploads/2012/07/Screenshot.png"><img src="http://0xdabbad00.com/wp-content/uploads/2012/07/Screenshot-300x65.png" alt="" title="Amazon EC2 Security Group setting" width="300" height="65" class="alignnone size-medium wp-image-507" /></a>
</ol>

<h3>OSX Client system Configuration</h3>
<ol>
<li>Install <a href="https://code.google.com/p/tunnelblick/">tunnelblick</a>.
<li>Copy the key file created previously to the config directory:
[sourcecode gutter="false"]
cd ~user/Library/Application\ Support/Tunnelblick/Configurations/
cp /tmp/openvpn-key.txt .
[/sourcecode]
<li>Create config file:
[sourcecode gutter="false"]
dev tun
proto udp
port 1194
remote server_address.com
resolv-retry infinite
nobind
secret openvpn-key.txt
ifconfig 10.8.0.2 10.8.0.1
comp-lzo
verb 8
redirect gateway def1
[/sourcecode]
Note the ifconfig line for the client and server are reversed.
</ol>

You can also use <http://www.sparklabs.com/viscosity/>viscosity</a> as opposed to Tunnelblick, but it costs money.  However, it is free for the first 30 days and it's error messages are more helpful if you have problems.

Everything should now be working, so go to a site that checks your IP from the OSX system and ensure it has changed.  May also want to run tcpdump to ensure all traffic is going to the EC2 IP.
