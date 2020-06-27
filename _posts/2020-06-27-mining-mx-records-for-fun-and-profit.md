---
layout: post
title: "Mining DNS MX Records for Fun and Profit"
description: ""
modified: 2020-06-27
tags: [security, research, dns, analytics, email, data science]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
---

If you have read my blog before, you may realize that I [really](http://www.covert.io/three-short-links-on-popular-domain-lists-for-threat-intelligence/) [love](http://www.covert.io/post/23331714509/hadoop-dns-mining) [DNS](http://www.covert.io/post/72223750985/dns-census-2013) [data](http://www.covert.io/six-short-links-on-pdns-graph-analytics-for-security/) and [dns](http://www.covert.io/getting-started-with-dga-research/) [analytics](http://www.covert.io/auxiliary-loss-optimization-for-hypothesis-augmentation-for-dga-domain-detection/).  In this post, I share some experiences in using mostly DNS data for identifying the visible footprint of popular email security providers.  

This may not be terribly novel, but it was an interesting exploration during a time of boredom for me.  This work was initially motivated by two events: 

1. When the Proofpoint email protection machine learning vulnerability ([CVE-2019-20634](https://nvd.nist.gov/vuln/detail/CVE-2019-20634)) was [announced by Will Pearce and Nick Landers](https://github.com/moohax/Talks/blob/master/slides/DerbyCon19.pdf) I got to wondering about how large their deployment footprint was and how one could figure this out, and 
2. A friend at another company mentioned that they were using a specific startup email security provider and I wondered whether I could determine what other companies were also using this same provider.

Here is the methodology I devised for this:

1. Collect a large sample of MX records
2. Enrich MX records with IP intelligence and useful metadata
2. Sift through the enriched records and identify recognizable email provider's domains through OSINT (whois, PDNS, Google) and market research.
4. Profit?!?!?

For step one, I downloaded the [Alexa top 1M domains](http://s3.amazonaws.com/alexa-static/top-1m.csv.zip), [Quantcast top 1m domains (from WaybackMachine)](https://web.archive.org/web/*/https://ak.quantcast.com/quantcast-top-sites.zip), [Domcop Top 10m domains](https://www.domcop.com/top-10-million-domains), [Majestic Million Domains](https://majestic.com/reports/majestic-million) and [Cisco Umbrella top 1m domains](https://umbrella.cisco.com/blog/cisco-umbrella-1-million).  I identified the registered domain using [tldextract](https://pypi.org/project/tldextract/) for each of these and then combined them into a [single de-duplicated list](https://mx-intel-public.s3.amazonaws.com/all-registered-domains.txt.gz).  This resulted in \~8.3M unique domain names.  I then performed bulk MX lookups using [adnshost](https://www.gnu.org/software/adns/) against my own bind9 recursive nameserver.  In my experience, adnshost works pretty well for bulk DNS resolution at this scale, and it will perform both the lookup requested (MX) as well as a domain resolution (A-lookup).  When performing bulk DNS lookups at this scale it is important to add retry logic for failed resolutions as this tends to happen enough to be a problem.  I did this using a [simple bash script](https://github.com/covert-labs/mx-intel/blob/master/robust_perform_resolutions.sh) that retried failed lookups up to three times.

For step two, I then developed a simple Jupyter notebook to parse the adnshost logs and perform the enrichments using [tldextract](https://pypi.org/project/tldextract/), PTR lookups (also using adnshost), Maxmind ASN, Maxmind City, Alexa ranking, and Cloud provider IP Ranges for [AWS](https://ip-ranges.amazonaws.com/ip-ranges.json), [Azure](https://www.microsoft.com/en-us/download/confirmation.aspx?id=56519), and [GCP](http://www.gstatic.com/ipranges/cloud.json).  **Side note**: I also attempted to perform SOA lookups on the /24 networks of each IP after noticing some useful patterns with failed PTR lookups.  This appears potentially useful for identifying some of uses of some of the IP space of the cloud providers, but this turned into a rabbit hole since adnshost appears to crash when trying to handle some of the results it received.

<img src="/images/mx-intel-diagram.png" border="1px">

For step three, I did the following:

1. Performed market research on the top email security providers as well as emerging and niche providers.  This [site](https://www.datanyze.com/market-share/email-security--343) was helpful as well as just googling around and exploring PDNS/Whois data from [PassiveTotal](https://community.riskiq.com/) and [SecurityTrails](https://securitytrails.com/).
2. Scrutinized the top MX server registered domains and ASNs and tried to identify potential security providers.
3. Sifted through the remaining results trying to identify any obvious providers with "malware", "phish", "spam", or "security" in their domain names.

I used this to build two mappings to email security providers: MX server base domains and ASN names.  The mappings can be found [here](https://github.com/covert-labs/mx-intel/blob/master/email_security_providers.py).  Then I summarized the overall dataset and those results are presented below.  **Elephants in the room** I purposefully did not include Microsoft, Google, and some of the bigger tech companies that provide email service as part of these mappings since I don't consider them email security companies.  This may be debatable since these companies do provide security features through their offerings.

### Brief Intro to MX records

For those of you who may not be familiar with DNS MX records, these are DNS Resource Records (RRs) used to map a domain name to the Mail Exchange (MX) servers responsible for accepting email for that domain.  MX records are used by Mail Transfer Agents (MTA) in order to identify where email should be sent for a given recipient email address.  Below we use the command line utility "dig" to perform an MX lookup on gmail.com to find its Mail Exchange servers.  As you can see, at the time of this writing, there are five MX domains that can accept email for gmail.com.  

<script src="https://gist.github.com/jatrost/dbe0c0b5b111fdb86e43a56a6b074e24.js"></script>

Besides being critical for identifying where email should be sent, MX records are also useful for mapping out infrastructure and can sometimes be used to identify which email security providers are being used by a company of interest.  Below is an example for Florida State University (go Noles!) that reveals that, at the time of this writing, they are using Proofpoint to receive their email.  How do we know this?  Their mail exchanges are hosted on sub domains of pphosted.com which is owned by Proofpoint.  

<script src="https://gist.github.com/jatrost/12d74b8837a84fc1865669a333ae75fa.js"></script>

Some companies obscure their security providers by first receiving their email to other mail exchanges such as ones hosted in their own data center or ones hosted by Google or Microsoft.  In this blog, we explore a large DNS dataset to identify interesting info about the visible footprint / market share of email security companies.  

All code and data for this study can be found in this Github Repo: [https://github.com/covert-labs/mx-intel](https://github.com/covert-labs/mx-intel).

## Observations:

* Email security provider OPSEC is remarkably bad in a lot of cases and it is often easy to determine which provider is being used.  Anyone who works in cyber security knows it is generally not a good idea to broadcast which cyber security products you are using since it may provide information that can be exploited by the adversary.  This is especially true when vulnerabilities are announced in security products.
* Since email exchanges can be chained together, only the outermost layer is visible in DNS MX records. For this reason, this research will underestimate the size of each provider's market share.
* Some security providers supply very specialized services (like anti-phishing only) and because of this they are often not the first layer in the email exchange chain.  They will be dramatically underrepresented in this study.

## Results:

### Summary: 

* 8,395,595 domains (derived from several top domain lists)
* 12,910,550 unique MX records (from 5994452 unique domains)
* 2,901,843 Unique Mail server domains
* 1,940,993 Unique Mail server base domains
* 25,733 Unique Mail server ASNs
* 56 Unique Security Providers identified

### Analytics:

Here are the questions I was hoping to answer with the tables presented below:

* Who are the market leading security companies reflected in the data?
* What is the visible market share of email security providers as reflected in DNS records?
* What can be inferred from publicly available MX records about email security?
* Which email security providers are leveraging cloud hosting?  And which cloud hosting environments are used most?
* Who are the visible customers of provider X?

Note: All tables below show the count of domains hosted, NOT companies; companies can own many domains.  Fortune 1000 domains are from 2015 and are based on [this file created by Bob Rudis](https://gist.github.com/hrbrmstr/ae574201af3de035c684/).

#### Top Email Security Providers Overall
<script src="https://gist.github.com/jatrost/61ff8d530c8dbb1587b5148beb890515.js"></script>

#### Fortune 1000 Email security providers
<script src="https://gist.github.com/jatrost/95914680804782b62d2273928575e730.js"></script>

#### Fortune 100 domain, MX base domain, email security provider

Note: I ended up adding Google and Microsoft to this table since they were very well represented.  As you can see, Proofpoint and self-hosting dominate the Fortune 100.

<script src="https://gist.github.com/jatrost/ffcea5afa0a204901c2bfefac931b30b.js"></script>

#### Alexa 1000 Email security providers
<script src="https://gist.github.com/jatrost/107d72575748e7f1e7975ce1f124bb8c.js"></script>

#### Alexa 100 domain, MX base domain, email security provider

Note: I ended up adding Google and Microsoft to this table since they were very well represented.  As you can see, self-hosting, Google and Microsoft dominate the Alexa 100.  Almost all of these domains are from large technology / web companies so this isn't so surprising, but it is interesting as compared to the Fortune 100.

<script src="https://gist.github.com/jatrost/df324355c19b0b1c0f1ca142d49f33e8.js"></script>

#### Top Email Security Providers Hosted in AWS

Many large email security companies are operating from AWS.

<script src="https://gist.github.com/jatrost/03a63988d2ffb1fdafa61355db2be1d9.js"></script>

#### Top Email Security Providers Hosted in Azure

Only a small number of identifiable email security companies were operating from Azure.

<script src="https://gist.github.com/jatrost/1683f4cd598c96f7787a2ff0fe1cd965.js"></script>

#### Top Self-hosted Email Security Providers
<script src="https://gist.github.com/jatrost/ec1a85077dbd9f6d171a82d9a7888711.js"></script>

## Misc Findings

When mining this data I discovered a few interesting items.

### Linode / CSC Digital Brand Services

One of the more popular email security providers, "CSC Digital Brand Services" (which service multiple Fortune 100 companies), uses Linode for their hosting. This was surprising since Linode seems like a much smaller player in the Cloud hosting market.

### googlemial[.]com 

When I initially collected this data, freecodecamp.org had a misconfigured MX domain pointing to googlemial[.]com.  And this sketchy domain is not owned by Google and resolved to a GCP IP.  Upon further inspection, this IP appears to be hosting a parking page for unregistered domains owned by GoDaddy.  A quick [PDNS check](https://securitytrails.com/list/ip/35.186.238.101) of other domains resolving to this IP reveals \~4.2M+ domains, and a quick DNS resolution on those domains with any subdomain shows that they all resolve to the same IP.

<script src="https://gist.github.com/jatrost/f5963734af7b7fe0f5bc0474cb5b6d17.js"></script>
**adnshost logs for freecodecamp.org**

## Future Work

I am not sure if I will return to this research or not, but I had some ideas that may be worth pursuing at some point, maybe during the next pandemic :)

* Perform similar work against a much larger scale - using all major zone files (COM, NET, ORG) and [ICANN's CZDS](https://czds.icann.org/home) as the inputs.
* Or perform similar work using the [Rapid7 Opendata](https://opendata.rapid7.com/) DNS data sets.
* Determine if port scans against MX servers could be useful to augment this.
* Automate PDNS queries and analysis against the MX records found to identify other domains not found in the top domain lists.
* Perform similar work, but collect SPF records and see what interesting insights could be gleaned about email sending trust (and whether vulns could be identified -- like AWS IPs in the SPF that are stale and potentially obtainable).
* Completely automate this entire process and use it to generate weekly reports.
* Identify providers hidden by the first layer mail exchange.  It may be possible to do this at scale (but only for some companies) if the companies send Bounced notifications to external email senders for non-existent recipients.  These bounced messages often contain all the SMTP headers of the original message sent.  These headers can reveal security products.  This technique was used on a targeted basis by Will Pearce and Nick Landers in their [DerbyCon research on Proofpoint](https://github.com/moohax/Talks/blob/master/slides/DerbyCon19.pdf).  Trying to do this at scale may draw a lot of attention or get my research box put on some blacklists.  It would also likely be a lot more effort to identify the SMTP headers associated with different security providers.

## Resources

Data:

* [all-registered-domains.txt.gz](https://mx-intel-public.s3.amazonaws.com/all-registered-domains.txt.gz) - base domains extracted from combining several popular domains lists together and then uniqued.
* [all-popular-domains-MX-20200620.txt.unique.gz](https://mx-intel-public.s3.amazonaws.com/all-popular-domains-MX-20200620.txt.unique.gz) - adnshost logs from performing MX lookups on domains from all-registered-domains.txt.gz.
* [mailserver_registered_domain-NS-20200620.txt.gz](https://mx-intel-public.s3.amazonaws.com/mailserver_registered_domain-NS-20200620.txt.gz) - adnshost logs from performing NS lookups on all the MX base domains; used for enrichment.
* [mx-intel-enriched.csv.gz](https://mx-intel-public.s3.amazonaws.com/mx-intel-enriched.csv.gz) - the final enriched output from this work.

Notebooks, Code, and summary results: [https://github.com/covert-labs/mx-intel](https://github.com/covert-labs/mx-intel).

--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)

