---
layout: post
title: "Borderless Threat Intelligence"
description: ""
modified: 2016-04-26
tags: [security, threat intelligence, r-cisc, brand monitoring]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
---

Borderless Threat Intelligence - using External Threat Intelligence for Brand and Supply Chain Monitoring

This past year was the year of the data breach.  Large and small organizations across every industry vertical were impacted by compromises that ranged from theft of PII, intellectual property, and financial information to publication of entire backend databases and email spools.

The data from these breaches often wound up being exposed publicly, exchanged or sold on underground markets, or simply leveraged to breach other organizations.

Many of these breaches have cascading effects due to the transitive nature of security that exists across companies, as the rely on critical business partners, subsidiaries and other organizations whose services are trusted.

Additionally, password reuse across customer accounts involved in a third-party data dump could enable unauthorized access to another business' assets.

Threat intelligence can be used to gain visibility and combat some of these issues. External threat intelligence can be leveraged to use information about events not occurring on your own network in order to detect breaches or threats to your brand or supply chains. Below are some specific items that can help:

## Suspicious Domain Name Monitoring

Adversaries are increasingly acquiring domain names that resemble either a company they want to target or a brand they want to leverage to social engineer victims. Registering typo squatted domains and homoglyph domains is not new and there are some great open source tools, such as [urlcrazy](http://www.morningstarsecurity.com/research/urlcrazy) and [dnstwist](https://github.com/elceef/dnstwist), to do this.

However, many organizations are not monitoring for the presence of these domains in their environment or the registrations of these domains as they occur.

Adversaries often use the following techniques when acquiring these domains -- here are some examples techniques applied to the domain "threatstream.com":

* Keyboard Typo – threststream.com
* Character omission – theatstream.com, threatsteam.com
* Character insertion – threattstream.com, threatsstream.com
* Homoglyph – threatstrearn.com, threatstreann.com
* Character Swap – threatstraem.com
* Missing dot or replacing dot with dash – wwwthreatstream.com, www-threatstream.com, http-www-threatstream.com, threatstreamcom.us
* Deceptive token insertion – login-threatstream.com, sslvpn-threatstream.com, support-threatstream.com
* Different TLD – threatstream.us, threatstream.co, threatstreamc.om


## Mass Credential Exposures

!["Mass Credential Exposures"](/images/credential-exposures.jpg)

Increasingly, script kids, hacktivists and cyber criminals are dumping huge collections of usernames and passwords publicly on the dark web or for sale on underground marketplaces. Often times, these dumps contain corporate email addresses and passwords that were found when a third-party website was compromised through SQL injection or other weaknesses.

Because password reuse is so rampant and many organizations still don't use Multi-Factor Authentication (MFA), these exposures can put you at risk. Monitoring for mass credential exposures affecting your company or any supply chain company you depend on is very useful for reducing these risks.

## Network Cleanliness Monitoring

Are IPs from the network space you own/control showing up on threat feeds of machines that are scanning, brute forcing, DDoS-ing, sending spam, connecting to DNS sinkholes, or hosting malicious content?

What about the IPs of your executive's home networks? Or the IPs of your critical supply chain or business partners?

If so, you may have a compromised system that you were unaware of and it may put your company at risk. Monitoring threat intelligence feeds for these items can give you a warning that something is not right so action can be taken.

!["Network Cleanliness Monitoring"](/images/scan-sites.jpg)

Have you checked what sites like [Shodan](https://www.shodan.io/), [ZoomEye](https://www.zoomeye.org/), and [Censys](https://censys.io/) have about your networks or your critical supply chains' networks? Are there vulnerable or unexpected services running? These sites' data is public so anyone, including malicious actors, can and do review it -- so should you.

Network cleanliness monitoring can take many shapes from monitoring threat intelligence feeds to querying public portscan/web crawl repositories to actually running the network scans yourself but the goals are to identify systems and services that are either vulnerable, unexpected, or compromised.

## Signs of Targeting and Social Networking Data-mining

Are you or your supply chain being discussed as a target on social media or the DarkWeb? Have any public threats been made? Is there malicious software purpose-built to target your company or your supply chain?

Monitoring what is being shared and discussed on both social media and the DarkWeb forums as it pertains to your company can provide an an early warning before an attack is carried out.

!["Signs of targeting and social networking data-mining"](/images/hell-forum.jpg)

Credential Exposure Posting from the Hell Darkweb forum

## Operationalizing

The first step for operationalizing these concepts is building an inventory of yourself and your critical supply partners. The following information should be collected and kept up-to-date:

* Email domains names
* Internal and external domain names, especially for sensitive resources
* Company’s IP address space
* Brand names
* Personal email addresses of key executives
* IP address space of key executives’ home networks
* Names of key executives

The next step is identifying data sources that you have or that you can acquire that provide the visibility you want. These data source should include:

* Internal DNS logs, web proxy logs
* Threat Intelligence feeds
  * Honeypot data
  * Malware C2 Sinkhole data
  * Spammer feeds
  * Portscan/web crawl data
* Paste sites (e.g. pastebin and others)
* DeepWeb/DarkWeb monitoring
* Social media sites
* Google/Bing searches

The final step is setting up collection and processing of the different data sources to look for instances of items from your inventory showing up. Security Information and Event Management (SIEM) or log management platforms are often useful places to perform these actions.

If you don't have one of these, simple scripts that grep the data may be sufficient (it really depends on the size of your network and the volume of data). There are commercial services that can provide this, as well.

If you would like to learn more about these concepts and how to operationalize them, please check out the presentation I gave at the 2016 R-CISC Retail Summit below.


<iframe src="//www.slideshare.net/slideshow/embed_code/key/nrImpegg8Bi0am" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> 
</iframe>
<div style="margin-bottom:5px"> 
<strong><a href="//www.slideshare.net/jasontrost/rcisc-summit-2016-borderless-threat-intelligence" title="R-CISC Summit 2016 Borderless Threat Intelligence" target="_blank">R-CISC Summit 2016 Borderless Threat Intelligence</a></strong> from <strong><a target="_blank" href="//www.slideshare.net/jasontrost">Jason Trost</a></strong> 
</div>