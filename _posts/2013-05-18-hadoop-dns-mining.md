---
layout: post
title: Hadoop DNS Mining
description: "Hadoop DNS Mining"
modified: 2013-05-18
tags: [security, machine learning, data science]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
permalink: /post/23331714509/hadoop-dns-mining
---

This is another quick post.  I have been working on this small framework for a while now and I decided to publish the code before I completely finished.

The hadoop-dns-mining framework enables large scale DNS lookups using Hadoop.  For example, if you had access to zone files from [COM, NET](http://www.verisigninc.com/en_US/products-and-services/domain-name-services/grow-your-domain-name-business/tld-zone-access/index.xhtml), [ORG](http://www.pir.org/help/access), etc (all free and publicly available), you could take each domain in these files and use this framework to resolve the domains for various record types (A, AAAA, MX, TXT, NS, etc).  After resolving the domains, you can run them through an enrichment process to add city, county, lat, long, ASN, and AS Name (via Maxmind's DBs).  And, you could scale out this collection and processing effort using Hadoop, say, running over EC2.

There are some interesting applications of this type of system, like using the existing zone files names to brute force "generating" zone files for TLDs that do not publish them (most ccTLDs do not).  For example, like this company has done: http://viewdns.info/data/

For more details check out my github repo.  The README covers DNS collection and geo enrichment.  I have code checked in that will store this data in [Accumulo](http://accumulo.apache.org/) using a few different storage/access patterns, but more explanation will come later as I have time.

https://github.com/jt6211/hadoop-dns-mining

--Jason<br />
[@jason_trost](https://twitter.com/#!/jason_trost)

