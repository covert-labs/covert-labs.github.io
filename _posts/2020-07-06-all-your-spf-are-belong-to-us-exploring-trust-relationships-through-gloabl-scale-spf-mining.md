---
layout: post
title: "All your SPF are belong to us: Exploring trust relationships through global scale SPF Mining"
description: ""
modified: 2020-07-06
tags: [security, research, dns, analytics, email, data science]
image:
  feature: alexa-100000-heatmap-header.png
comments: true
share: true
---

In this post we explore a large collection of Sender Policy Framework (SPF) records to see what they might tell us about global email sending trust relationships and how they relate to email security providers.  This is a fast follow-up to my previous post on [Mining DNS MX Records for Fun and Profit](http://www.covert.io/mining-mx-records-for-fun-and-profit/).  

Here is the methodology I devised for this (very similar to the previous post, but with [new](https://github.com/covert-labs/mx-intel/blob/master/parallel_dig.sh) [custom](https://github.com/covert-labs/mx-intel/blob/master/spf_crawler.py) [built](https://github.com/covert-labs/mx-intel/blob/master/spf_results_parser.py) [tools](https://github.com/covert-labs/mx-intel/blob/master/SPF-Parse-Enrich.ipynb)):

1. Collect a large sample of SPF records via DNS TXT lookups of popular domain names (and recursively resolving SPF "include" domains).
2. Enrich SPF records with IP intelligence and useful metadata (including [email security provider mappings](https://github.com/covert-labs/mx-intel/blob/master/email_security_providers.py))
3. Analyze the enriched results.

## Intro to Sender Policy Framework (SPF)

The Sender Policy Framework (SPF) enables domain name administrators to authorize hosts to use their domain names when sending email (i.e. in the "MAIL FROM" or "HELO" identities in SMTP).  SPF records are published using DNS TXT records.  SPF compliant mail receivers use the published SPF records to test the authorization of sending Mail Transfer Agents (MTAs).  SPF can be used to build complex policies around who can send email on whose behalf.  Below is an example SPF record for Florida State University.

<script src="https://gist.github.com/jatrost/544dfbc979332f6948a2bca065830dc5.js"></script>

According to this SPF record 146.201.58.212, 146.201.58.213, 146.201.107.145, 146.201.107.249, 192.12.121.23, and 199.188.157.80 are allowed to send email purporting to be from fsu.edu.  Also, the SPF records from spf.protection.outlook.com, \_spf.qualtrics.com, spf.blackboardconnect.com, servers.mcsv.net, and \_spf.mlsend.com should be retrieved and their policies applied as well.  Below are the SPF records for each of these domains.  As you can see they include more and more IPs/CIDRs as well as additional SPF includes.  

<script src="https://gist.github.com/jatrost/e342a1b77bde98d231cc4ef3f30e71b7.js"></script>

As you can see, SPF forms a chain of trust between the domain owner and all the SPF policies included recursively (potentially crossing several different administrative boundaries).  In this post I was hoping to explore this chain of trust at a large scale by collecting a large sample of SPF records and mining them.  

Below are some useful resources for understanding SPF:

* [RFC7208: Sender Policy Framework (SPF) for Authorizing Use of Domains in Email](https://tools.ietf.org/html/rfc7208)
* [SPF Syntax Table](https://dmarcian.com/spf-syntax-table/) - really useful guide for understanding SPF "mechanisms".

## Step One: Collection

For step one, I built a very ~~crude~~ useful [SPF crawler](https://github.com/covert-labs/mx-intel/blob/master/spf_crawler.py) that uses dig (optionally adnshost) to perform DNS TXT requests, parse out SPF records found, and then recursively follow the trail of SPF include records and perform TXT lookups against the included domains.

In order to seed the SPF crawler, I used the same domains I used in my [previous blog post on mining MX records](http://www.covert.io/mining-mx-records-for-fun-and-profit/).  I downloaded the [Alexa top 1M domains](http://s3.amazonaws.com/alexa-static/top-1m.csv.zip), [Quantcast top 1m domains (from WaybackMachine)](https://web.archive.org/web/*/https://ak.quantcast.com/quantcast-top-sites.zip), [Domcop Top 10m domains](https://www.domcop.com/top-10-million-domains), [Majestic Million Domains](https://majestic.com/reports/majestic-million) and [Cisco Umbrella top 1m domains](https://umbrella.cisco.com/blog/cisco-umbrella-1-million).  I identified the registered domain using [tldextract](https://pypi.org/project/tldextract/) for each of these and then combined them into a [single de-duplicated list](https://mx-intel-public.s3.amazonaws.com/all-registered-domains.txt.gz).  This resulted in \~8.3M unique domain names.

These domains were fed into my SPF crawler and then the results were collected, parsed, and then assembled.  I ended up backing the SPF crawler with "dig" instead of "adnshost" this time since I found dig was more reliable, completing 23% more DNS requests in an experiment against the Fortune 1000 domains.  Dig is single threaded, but I easily parallelized it using splits files and xargs and its performance ended up being good enough.  See [parallel_dig.sh](https://github.com/covert-labs/mx-intel/blob/master/parallel_dig.sh) for more details.

Below are a few simple commands as well as example output data collected with my SPF crawler applied to just one domain.  As you can see, the assembled output for fsu.edu includes all the IPs and Netblocks from all the SPF includes that it links to, recursively.

<script src="https://gist.github.com/jatrost/ea5826ea2d5596499507e5ad78bec398.js"></script>

Below is the same information, visualized as a network (and enriched with ASN info from Maxmind).

<a href="/images/spf/fsu-networkx.png"><img src="/images/spf/fsu-networkx.png" width="600px"></a>

## Step Two: Enrichment

For this step, I reused [a lot of the code](https://github.com/covert-labs/mx-intel) from my previous blog post on [Mining MX records](http://www.covert.io/mining-mx-records-for-fun-and-profit/) and performed the following enrichments:

1. Maxmind ASN
2. Maxmind Country
3. Cloud Provider IP Lookups for AWS, Azure, and GCP
4. Alexa Ranking
5. [Email Security Provider mapping](https://github.com/covert-labs/mx-intel/blob/master/email_security_providers.py)

[netaddr](https://pypi.org/project/netaddr/), [tldextract](https://pypi.org/project/tldextract/), and [cidr-trie](https://github.com/Figglewatts/cidr-trie) were useful during this stage.

## Step Three: Analysis

Through this analysis, I hoped to answer the following questions:

* What is the largest trusted network size (both single CIDR and aggregate network space)? ... HUGE
* Could I find any blatantly misconfigured SPF records? ... YES
* What does SPF data show about email security providers? ... A lot that MX doesn't
* What are the most "included" SPF includes?  ... Not many surprises here
* Does SPF augment the MX record mining (give more coverage? reveal things previously hidden? or 100% redundant?) ... YES!
* Are domains trusting IP space from cloud providers that may be re-usable (i.e. AWS EC2)? ... YES!

Below are some outputs and commentary from this project's Jupyter notebook that answer the questions above.

These [networkx](https://networkx.github.io/) visualizations are a bit of a mess, but they should get the point across of how interconnected the SPF trust relationships are.

### Fortune 100 SPF Trusted Networks Graph
<a href="/images/spf/fortune-100-networkx.png"><img src="/images/spf/fortune-100-networkx.png" width="600px"></a>

### Alexa 100 SPF Trusted Networks Graph
<a href="/images/spf/alexa-100-networkx.png"><img src="/images/spf/alexa-100-networkx.png" width="600px"></a>

## Heatmaps

As you can see from the next several heatmaps, as we go beyond the Alexa top 1,000 domains the number of networks trusted drastically increases, and as we hit the Alexa 1m, the entire Internet is trusted.  

These heatmaps were generated with the awesome [ipv4-heatmap](https://github.com/measurement-factory/ipv4-heatmap) tool provided by the [Measurement Factory](http://www.measurement-factory.com/).  The code to automate this can be found in my Jupyter Notebook [here](https://github.com/covert-labs/mx-intel/blob/master/SPF-Parse-Enrich.ipynb).

### Fortune 1,000 SPF Trusted Networks Heatmap
<img src="/images/spf/fortune-1000-heatmap.png" width="600px">

### Alexa 1,000 SPF Trusted Networks Heatmap
<img src="/images/spf/alexa-1000-heatmap.png" width="600px">

### Alexa 10,000 SPF Trusted Networks Heatmap
<img src="/images/spf/alexa-10000-heatmap.png" width="600px">

### Alexa 100,000 SPF Trusted Networks Heatmap
<img src="/images/spf/alexa-100000-heatmap.png" width="600px">

### Alexa 1,000,000 SPF Trusted Networks Heatmap
<img src="/images/spf/alexa-1000000-heatmap.png" width="600px">

### Alexa Top 1M Domains Trusting /7 or larger networks

As you can see from this list, there are quite a few domains that trust very large networks.  Several of these seem like likely misconfigurations. For example, these four domains trust the entire Internet:

* hitadouble[.]com: 208.67.207.0/0
* payukraine[.]com: 0.0.0.0/0
* angliss[.]edu[.]au: 0.0.0.0/0
* hutkigrosh[.]by: 0.0.0.0/0

This domain trusts half of the Internet -  salaam[.]af: 175.106.32.0/1

And these five domains trust 1/4 of the Internet.  cfe[.]fr appears to have fixed this apparent misconfiguration now.  As their TXT record has changed.

* creativecircle[.]com: 64.4.22.64/2
* gevestor[.]de: 91.241.72.0/2
* debeersgroup[.]com: 10.47.149.168/2
* cfe[.]fr: 82.97.62.0/2
* adecco[.]com: 148.105.8.0/2

<script src="https://gist.github.com/jatrost/1bb8d1fa91e2346cb1deca6a6e7761e6.js"></script>

### Top SPF Includes from all top domain lists (via SPF)
<script src="https://gist.github.com/jatrost/4d60851dcab2928e5e82e68714187237.js"></script>

Using all the popular domain names, here is a summary of the top 10 SPF includes. 

Major Cloud Email Providers:
* Microsoft: spf.protection.outlook.com 
* Google: \_spf.google.com   

Hosting Providers:
* HostGator: websitewelcome.com 
* OVH: mx.ovh.com 
* Bluehost: bluehost.com    

Commercial Email Marketing companies
* MailChimp: servers.mcsv.net
* Mandrill: spf.mandrillapp.com (MailChimp add-on)
* Sendgrid: sendgrid.net

Email Security company: 
* MailChannels: mailchannels.net (more on this later)

### Top SPF Includes from Fortune 1000 (via SPF)
<script src="https://gist.github.com/jatrost/6944e81b2759125be864dde4f3db4ced.js"></script>

### Top SPF Includes from Alexa top1m
<script src="https://gist.github.com/jatrost/b8c209b7149768123e357301a66b89e6.js"></script>

## Email Security Providers

If you read my previous blog post on [Mining DNS MX Records for Fun and Profit](http://www.covert.io/mining-mx-records-for-fun-and-profit/), then you might notice that these top lists look significantly different than the top email providers as identified from MX records.  The top 5 providers identified in the SPF data are MailChannels, Mimecast, Proofpoint, Solarwinds, and Barracuda.  In the MX post, the top 5 were Proofpoint, Mimecast, Deteque, Barracuda, and Solarwinds, AND MailChannels was #48 on that list.  These top lists are using all the popular domains data which is likely not an accurate reflection of the actual email security market.  When reviewing the Fortune 1000 top Email Security providers the story is not as surprising as the top 4 from the Fortune 1000 Email security providers were nearly identical across SPF and MX records with just the order being different.  I suspect that MailChannels shows up as popular in SPF because either it is the default setting on newly registered domains OR it is the default setting for domains that are parked with certain hosting providers, but I haven't spent the time to prove/disprove this.

One other interesting aspect with SPF is it (potentially) reveals relationships with multiple email security providers.  See the "Fortune 100 Email Security Providers Listing (via SPF)" and "Domains with 4 or more Email Security Providers (via SPF)" gists below.  In the Fortune 100 list, there are 3 domains with SPF relationships with more than one provider.  If you look across all the top domains data you can see there are many.  For anyone who has worked in the cyber security department at a large company before, this is not surprising, but it was cool to be able to see this in the data.

* Domains with 2 SPF relationships with Email Security Providers: 11,393
* Domains with 3 SPF relationships with Email Security Providers: 468
* Domains with 4 SPF relationships with Email Security Providers: 35
* Domains with 5 SPF relationships with Email Security Providers: 1

###  Top Email Security Provider from all top domain lists (via SPF)
<script src="https://gist.github.com/jatrost/a1a3a0b2c4a7dbc3babefee99ad753eb.js"></script>

### Top Email Security Provider from Alexa 1m (via SPF)
<script src="https://gist.github.com/jatrost/657214251b56e6267059330a169b0ea1.js"></script>

### Top Email Security Provider from Fortune 1000 (via SPF)
<script src="https://gist.github.com/jatrost/d16bf6b4dc95e1c122b88593a3803e05.js"></script>

### Top Email Security Provider from Fortune 100 (via SPF)
<script src="https://gist.github.com/jatrost/56344ec6076379a83958087ae63afc3e.js"></script>

### Fortune 100 Email Security Providers Listing (via SPF)
<script src="https://gist.github.com/jatrost/34a30bca216d171ae66f466359e498a5.js"></script>

### Domains with 4 or more Email Security Providers (via SPF)
<script src="https://gist.github.com/jatrost/4b505a6dbcbbd55e7e068cf7262fb468.js"></script>

## Trusting Cloud Provider Networks

As you can see from the next few tables, many domains transitively trust a lot of Cloud provider IP space for SPF.  For some of the larger networks trusted it seems like this carries risk since it may be possible for the cloud IP space to get reused; see [Fishing the AWS IP Pool for Dangling Domains
](https://labs.bishopfox.com/tech-blog/2015/10/fishing-the-aws-ip-pool-for-dangling-domains) for a practical example of this. 

Alexa 1000 Trusting AWS Networks
<script src="https://gist.github.com/jatrost/9349839085ad5362d9cbb8ae981da524.js"></script>

Alexa 1000 Trusting Azure Networks
<script src="https://gist.github.com/jatrost/3fc802f5727049f390f427c5cc43651c.js"></script>

Alexa 1000 Trusting GCP Networks
<script src="https://gist.github.com/jatrost/e956f69d3d8120898eba5f2b07c19dac.js"></script>

Fortune 1000 Trusting AWS Networks
<script src="https://gist.github.com/jatrost/6b2ce9608a2ec3078243bd2dcdc99cc0.js"></script>

Fortune 1000 Trusting Azure Networks
<script src="https://gist.github.com/jatrost/44eedaa273e05522953581b09d4e5c1d.js"></script>

Fortune 1000 Trusting GCP Networks
<script src="https://gist.github.com/jatrost/a240ba87eb430cecfe62fe41d5ba752a.js"></script>

### Some other potentially interesting results, not worth dumping here:

* [Alexa top1m domains trusting AWS Networks](https://gist.github.com/jatrost/60cc44bf1b3b4a4617ca8ffb74b726a7)
* [Alexa top1m domains trusting Azure Networks](https://gist.github.com/jatrost/97214849df789fd987b585f7321a8907)
* [Alexa top1m domains trusting GCP Networks](https://gist.github.com/jatrost/a9b3c5ed9efd6cdd931673d9da6882e1)
* [Top Maxmind ASNs of SFP Trusted Networks from Fortune 1000 (via SPF)](https://gist.github.com/jatrost/6d789a41a0712b1ef71818234335d5eb)
* [Top Maxmind ASNs of SFP Trusted Networks from all top domain lists (via SPF)](https://gist.github.com/jatrost/eb52a7f3c19607ce4d08407660fc09aa)
* [Top Maxmind ASNs of SFP Trusted Networks from Alexa top1m (via SPF)](https://gist.github.com/jatrost/f2e3567eb48788b8ba923852cb4aec96)
* Graph analytics applied to Fortune 1000 and Alexa 1000: degree centrality, edge betweenness centrality, pagerank, closeness centrality, triangle counts, and connected components stats, see the [notebook](https://github.com/covert-labs/mx-intel/blob/master/SPF-Parse-Enrich.ipynb) and search for "print_graph_metrics".

### Future Work

* SPF Crawler enhancements:  As you can see from the SPF guide I shared above for ["a"](https://dmarcian.com/spf-syntax-table/#a) and ["mx"](https://dmarcian.com/spf-syntax-table/#mx), SPF supports some fairly complex policies for allowing certain IPs to send email (esp the prefix operators on these SPF mechanisms).  I did not provide support for these mechanisms in the first version of my SPF crawler mainly due to the complexity involved.  Because of this, my results will under represent the trust relationships where these are used.  I hope to add support for these operators to expand what could be found in this data.
* Try some more graph analytics on the entire dataset.  In the Jupyter notebook I ran several graph algorithms on subsets of the entire graph (Fortune 100 and Alexa 100).  These showed some mildly interesting results, but testing against larger graphs caused graphviz to fail due to some data format issues that I have not had a chance to research.

### Resources

Notebooks, Code, and summary results: [https://github.com/covert-labs/mx-intel](https://github.com/covert-labs/mx-intel).

#### Data:

* [all-registered-domains.txt.gz](https://mx-intel-public.s3.amazonaws.com/all-registered-domains.txt.gz) - base domains extracted from combining several popular domains lists together and then uniqued.
* [spf-results-all-registered-domains.json.gz](https://mx-intel-public.s3.amazonaws.com/spf-results-all-registered-domains.json.gz) - the raw results from running the SPF Crawler against all-registered-domains.txt.gz.
* [spf-linked-all-registered-domains.json.gz](https://mx-intel-public.s3.amazonaws.com/spf-results-all-registered-domains.json.gz) - the assembled results from processing spf-results-all-registered-domains.json.gz.  This is the collapsed/combined data that shows all the SPF domains and networks included recursively.

--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)


