---
layout: post
title: Port Knocking! How to enable in Linux
excerpt: Here I'm going to explain one simple iptables configuration to enable port knocking which can protect your server from un-authorized access.
---

## Introduction

[<img align='right' class="size-medium wp-image-320 alignright" style="margin-left: 20px; margin-right: 20px;" alt="Port Knocking" src="http://www.boynux.com/wp-content/uploads/2013/08/ID-10011403-198x300.jpg" width="198" height="300" />][1] 
Security in Linux servers and desktops has a very important role, when it comes to security what's immediately comes into mind is the Internet and network security. If your box is connected directly to the public Internet or Intranet network, it means that you don't have much control over what malicious attackers can send to your network, what you can do is to protected and prevent all these threats to reach your servers and create bullet proof security guard to to be protected. 

In recent years many different systems including hardware and software have developed to help enterprises protect their assets and data from attackers and all those systems are doing well, but not always an expensive enterprise solution is the best choice, sometimes a fairly simple configuration in your server is enough to protect your from some attacks. 

Here I'm going to explain one simple iptables configuration which can protect your server and desktop ports from DDoS or Brute-force attack to some extent. 

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="horizontal"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

## Scenario 

Assume that you have a server with SSH service enabled, and you need to access your server from the Internet network. you have a good iptables rules enabled to protected all your other ports of server as well. 

Suddenly you notice that you get lots of failed SSH log-in requests to your server and it's just annoying. What quickly comes up in sysadmins mind is something like fail2ban script or similar in order to analyze logs and create proper iptables rules to ban malicious IPs from attacking server.  Maybe for a predefined time frame. 

Yes it's a good solution, but is there any way to be able to show our SSH port (22) closed or filtered to the public network, but our sysadmins be able to connect to it? With this trick we can avoid any attacks because actually the SSH port is not open! 

## How it works 

Many of you may already be familiar with port-knocking technique but for those who are new to this concept I'll explain more. 

What we do is to drop all requests to our destination port (in this scenario SSH port). So from network view this port is actually filtered. But how we can use it to connect to SSH or any other service (it actually could be HTTP, Telnet or any other service)? 

We first knock the door, and it means that we send a sequence of predefined packets to server which could be a simple `ICMP` (`ping`), `UDP`, `TCP` or any combination or number of these packets. When server (iptables) receives this sequence and it matches to it's definitions it opens the door (SSH in our case) and we can connect to server as usual. 

For this example I'll send one `TCP` packet to port 1000 to open the door, you can define to send multiple packets or one packet to port 1000 and one ICMP to other port or any other secret combination you like. 

## Interested? Here is how to do it.

Note: in order to use this rules you need to have iptables' `recent` kernel modules in your system. However in most recent servers it's available in default installation. 

	-A INPUT -p tcp -m state --state RELATED,ESTABLISHED -j ACCEPT 

This rule is not really part of port knocking and you may have it already in your rules. But in order to avoid dropping SSH packets after the connection has been established this rules is required, just ignore it if you already have it. 

<div class="ads"> <ins class="adsbygoogle adslot_1" style="display:block" data-ad-client="ca-pub-7360583392867579" data-ad-slot="4587256441" data-ad-format="rectangle"></ins> <script> (adsbygoogle = window.adsbygoogle || []).push({}); </script> </div>

	-A INPUT -p tcp -m tcp --dport 1000 -m recent --set --name KNOCKING --rsource
	-A INPUT -p tcp -m tcp --dport 22 -m recent --rcheck --seconds 5 --name KNOCKING --rsource -j ACCEPT
	-A INPUT -p tcp -m tcp --dport 22 -j DROP

A little more explanation about above rules: 

The first rule checks for `TCP` packets to destination port 1000. If it detects the packet, it sets a recent name called `KNOCKING` to recent table. `--rsource` means it keeps track of Remote Source, which we are interested in. The second rule checks recent table for the `KNOCKING` with `--rcheck` option and also checks for 5 seconds timeout of record. 

It means that `SSH` request has to be initiated withing 5 seconds after knocking port 1000, and if this rule matches, it just allows SSH packets to pass. The last rule is simply drops any `SSH` request (ie. port 22 `TCP` packets). 

Very simple, huh? Now if you add those rules in iptables and try to `SSH` to your box it simply does not allow you! So, you need to initiate the following command instead, in order to be able to SSH to server: 

	telnet <server ip> 1000 ; ssh <server ip>

Voilà, That's all! Nobody is allowed to enter without knocking the door :) 

I hope this could be useful and interesting for you. 

## Resources: 

+ <a href="http://linux.die.net/man/8/iptables" target="_blank">http://linux.die.net/man/8/iptables</a> 
+ <a href="http://www.netfilter.org/projects/iptables/" target="_blank">http://www.netfilter.org/projects/iptables/</a> 
+ <a href="http://www.netfilter.org/projects/iptables/" target="_blank">https://help.ubuntu.com/community/IptablesHowTo</a> 
+ <a href="http://www.netfilter.org/documentation/HOWTO/netfilter-extensions-HOWTO-3.html#ss3.16" target="_blank">http://www.netfilter.org/documentation/HOWTO/netfilter-extensions-HOWTO-3.html#ss3.16</a> 
+ <a href="http://blog.zioup.org/2008/iptables_recent/" target="_blank">http://blog.zioup.org/2008/iptables_recent/</a> 

Update 28-08-2013: Peter (from LinkedIn network) kindly shared these two useful resources, so I listed them here if you are interested in port knocking and read all the way to here, you may find these very interesting as well: 

+ <a href="http://www.microhowto.info/howto/implement_port_knocking_using_iptables.html " target="_blank">A sequential port knocking implementation using iptables</a>
+ <a href="http://www.zeroflux.org/projects/knock/" target="_blank">`knockd` project, which is actually an interface to iptables to implement port knocking</a> 

[1]: http://www.boynux.com/wp-content/uploads/2013/08/ID-10011403.jpg
