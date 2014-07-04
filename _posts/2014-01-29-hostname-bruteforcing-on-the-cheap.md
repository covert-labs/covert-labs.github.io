---
layout: post
title: Hostname bruteforcing on the cheap - Room362.com
description: "Hostname bruteforcing on the cheap from @mubix"
modified: 2014-01-29
tags: [security, dns, brute force, pentest]
image:
  feature: abstract-1.jpg
comments: true
share: true
---

Mubix ([@mubix](https://twitter.com/mubix)) [used](http://www.room362.com/blog/2014/01/29/hostname-bruteforcing-on-the-cheap/) the subdomains list from [here](http://www.ethicalhack3r.co.uk/zone-transfers-on-the-alexa-top-1-million/) to bruteforce subdomains using dig and args.  Really nice use of xargs for parallel execution.

    cat subdomains.txt | \
    	xargs -P 122 -I subdomain dig +noall subdomain.microsoft.com +answer


--Jason
