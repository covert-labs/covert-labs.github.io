---
layout: post
title: Internet Scale Port Scan Data and Analysis
description: "Internet Scale Port Scan Data and Analysis"
modified: 2013-03-18
tags: [portscan, internet scale, big data, security, hacking]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
permalink: /post/45716321979/internet-scale-port-scan-data-and-analysis
---

![](/images/internet-census-world-map.png)

A couple days ago, this was posted:

<blockquote>
<div><a href="http://internetcensus2012.bitbucket.org/paper.html">Port scanning /0 using insecure embedded devices</a><br /><br />

Abstract While playing around with the Nmap Scripting Engine (NSE) we discovered an amazing number of open embedded devices on the Internet. Many of them are based on Linux and allow login to standard BusyBox with empty or default credentials. We used these devices to build a distributed port scanner to scan all IPv4 addresses. These scans include service probes for the most common ports, ICMP ping, reverse DNS and SYN scans. We analyzed some of the data to get an estimation of the IP address usage. 
</div></blockquote>

It is a write up about performing an Internet scale port scan using thousands of compromised busybox embedded devices/linux servers.

While this is wildly unethical, and almost certainly illegal, the results of this study are pretty interesting and it is more interesting that the author decided to post all his code and data (~9TB uncompressed, 1.5 TB Compressed) online for free downloads.

The author also posted some interactive web apps that allow exploration of this data set:

![](/images/internet-census-ipv4-heatmap.png)

* [Browsable Hilbert Curve Maps](http://internetcensus2012.bitbucket.org/hilbert.html)
* [Service Probe Data](http://internetcensus2012.bitbucket.org/serviceprobe_overview.html)
* [Reverse DNS Exploration](http://internetcensus2012.bitbucket.org/tld_overview.html)

It is definitely interesting to see how [more](http://punkspider.hyperiongray.com/) and [more](http://www.shodanhq.com/) network/security data is being collected and made available freely on the Internet.  I am undecided whether this helps security or hurts security longterm.  It definitely makes the situation worse i the short term.

--Jason