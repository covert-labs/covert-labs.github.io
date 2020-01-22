---
layout: post
title: "Collecting and Curating IOC Whitelists for Threat Intelligence and Machine Learning Research"
description: ""
modified: 2020-02-01
tags: [security, research, machine learning, DNS, whitelists, training data]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
---

In this post, I share my experience in building and maintaining large collections of benign IOCs (whitelists) for Threat Intelligence and Machine Learning Research.

Whitelisting is a useful concept in Threat Intelligence correlation since it can be very easy for benign observables to make their way into [threat intelligence indicator feeds](/threat-intelligence/), esp. coming from open source providers or vendors that are not as careful as they should be.  If these threat intelligence feeds are used for blocking (e.g. in firewalls or WAF devices) or alerting (e.g. log correlation in SIEM or IDS), the cost of benign entries making their way into a security control will be very high (wasted analyst time for triaging false positive alerts or loss of business productivity for blocked legitimate websites).  Whitelists are generally used to filter out observables from threat intelligence feeds that almost certainly would be marked as a false positive if they were intersected against event logs (e.g. bluecoat proxy logs, firewall logs, etc) and used for alerting. 

Whitelists are generally used for:

* Filtering out observables from threat intelligence lists before correlation with event data
* Building labeled datasets useful for building machine learning models.
* Enriching threat intelligence or alerts with contextual information.

The classic example of this is 8.8.8.8 (Google's published open DNS resolver).  This has found its way into many open source and commercial threat intelligence feeds by mistake since sometimes malware use this IP for DNS resolution or they ping it for connectivity checks.  There are many other observables that commonly make their way into threat feeds due to how the threat feeds are derived / collected.  Below are a summary of the major sources of false positives for threat intelligence feeds and ways to identify these to prevent their use.  Most commercial threat intelligence platforms are pretty good at identifying these today and the dominant open source threat intelligence platform MISP is getting better with its [MISP-warninglists](https://github.com/MISP/misp-warninglists/), but as you will discover below there is some room for improvement.

## Benign Inbound Observables

Benign Inbound Observables commonly show up in threat intelligence feeds derived from distributed network sensors such as honeypots or firewall logs.  These IPs show up in firewall logs and are generally benign or at best are considered noise.  Below are several common Benign Inbound Observable types. Each type also comes with recommended data sources or collection techniques listed as sub bullets:

* **Known Web Crawlers** - Web crawlers are servers that crawl the World Wide Web and through this process may enter the networks of many companies or may accidentally hit honeypots or firewalls.
    * RDNS + DNS analytics can be used to enumerate these in bulk.  Example for google https://support.google.com/webmasters/answer/80553?hl=en.  Mining large collections of rdns data can reveal other patterns to focus on.  Below is an example of a simple PTR lookup on a known googlebot IP.  This should start to reveal patterns that could be codified assuming you have access to a large corpus of RDNS data like is provided here https://opendata.rapid7.com/sonar.rdns_v2/ (or could easily be generated).  

```
$ dig +nocomments +nocmd +noidentify +noquestion +nostats -x 66.249.66.1
1.66.249.66.in-addr.arpa. 86400    IN    PTR    crawl-66-249-66-1.googlebot.com.
$ dig +nocomments +nocmd +noidentify +noquestion +nostats crawl-66-249-66-1.googlebot.com
crawl-66-249-66-1.googlebot.com. 60 IN    A    66.249.66.1
```

* **Known port scanners** associated with highly visible projects or security companies (shodan, censys, etc)
    * RDNS + DNS analytics may be able to enumerate these in bulk (assuming the vendors want to be identified).  Example:

```
$ dig +nocomments +nocmd +noidentify +noquestion +nostats -x 198.20.69.98
98.69.20.198.in-addr.ARPA. 3320    IN    PTR    census2.shodan.io.

$ dig +nocomments +nocmd +noidentify +noquestion +nostats census2.shodan.io
census2.shodan.io.    300    IN    A    198.20.69.98
```

* **Mail Sending Servers** - these servers send email and they sometimes wind up on Threat feeds by mistake.  
    * In order to enumerate these, you need a good list of popular email domains.  Then perform DNS TXT request against this list and parse the SPF records.  Multiple lookups will likely be needed as SPF allows for redirects and includes.  Below shows the commands needed to do this manually for gmail.com as an example.  The CIDR blocks returned are the IP space where gmail emails are sent from.  Alerting or blocking on these is gonna cause a bad day.

```
$ dig +nocomments +nocmd +noidentify +noquestion +nostats TXT gmail.com
gmail.com.        300    IN    TXT    "v=spf1 redirect=_spf.google.com"
gmail.com.        300    IN    TXT    "globalsign-smime-dv=CDYX+XFHUw2wml6/Gb8+59BsH31KzUr6c1l2BPvqKX8="

$ dig +nocomments +nocmd +noidentify +noquestion +nostats TXT _spf.google.com
_spf.google.com.    300    IN    TXT    "v=spf1 include:_netblocks.google.com include:_netblocks2.google.com include:_netblocks3.google.com ~all"

$ dig +nocomments +nocmd +noidentify +noquestion +nostats TXT _netblocks.google.com
_netblocks.google.com.    894    IN    TXT    "v=spf1 ip4:35.190.247.0/24 ip4:64.233.160.0/19 ip4:66.102.0.0/20 ip4:66.249.80.0/20 ip4:72.14.192.0/18 ip4:74.125.0.0/16 ip4:108.177.8.0/21 ip4:173.194.0.0/16 ip4:209.85.128.0/17 ip4:216.58.192.0/19 ip4:216.239.32.0/19 ~all"

$ dig +nocomments +nocmd +noidentify +noquestion +nostats TXT _netblocks2.google.com
_netblocks2.google.com.    666    IN    TXT    "v=spf1 ip6:2001:4860:4000::/36 ip6:2404:6800:4000::/36 ip6:2607:f8b0:4000::/36 ip6:2800:3f0:4000::/36 ip6:2a00:1450:4000::/36 ip6:2c0f:fb50:4000::/36 ~all"

$ dig +nocomments +nocmd +noidentify +noquestion +nostats TXT _netblocks3.google.com
_netblocks3.google.com.    3187    IN    TXT    "v=spf1 ip4:172.217.0.0/19 ip4:172.217.32.0/20 ip4:172.217.128.0/19 ip4:172.217.160.0/20 ip4:172.217.192.0/19 ip4:172.253.56.0/21 ip4:172.253.112.0/20 ip4:108.177.96.0/19 ip4:35.191.0.0/16 ip4:130.211.0.0/22 ~all"
```


* **Cloud PaaS Providers** – Most Cloud providers publish their IP space via APIs or their documentation.  These lists are useful to derive whitelists, but they will need to be further filtered.  Ideally you only whitelist IP space that are massively shared (like S3, CLOUDFRONT, etc), not IPs that are easy for bad guys to use like EC2s.  These whitelists should not be used to exclude domain names point to this IP space, but instead should be used for either enrichments on alerting or to suppress IOC based alerting from these IP ranges.
    * Amazon AWS https://ip-ranges.amazonaws.com/ip-ranges.json 
    * Google Cloud Platform https://gist.github.com/n0531m/f3714f6ad6ef738a3b0a   
    * Azure https://www.microsoft.com/en-us/download/details.aspx?id=56519

**Note**: https://greynoise.io/ is commercial provider of "anti-threat" intelligence (i.e. they identify the noise and other benign observables), and they are very good at identifying the types of benign observables listed above.

**Note**: [MISP-warninglists](https://github.com/MISP/misp-warninglists) provides a lot of these items today but they may be stale (several of their lists have not been updated in months).  Ideally all of these lists are kept up-to-date through automated collection from authoritative sources instead of hard coded data stored in github (unless these are automatically updated frequently).  See section on "Building / Maintaining Whitelist Data” for more tips.

## Benign Outbound Observables

Benign Outbound Observables typically show up frequently in threat intelligence feeds related to malware sandboxing, URL sandboxing, outbound web crawling, email sandboxing, and other similar threat feeds.   Below are several common Benign Outbound Observable types. Each type also comes with recommended data sources or collection techniques listed as sub bullets:

* **Popular Domains** - Popular domains can wind up on threat intelligence feeds, especially those derived from malware sandboxing since often times malware uses benign domains as connectivity checks and some malware, like those conducting click fraud act more like web crawlers, visiting many different benign sites.  These same popular domains show up very often in most corporate networks and are almost always benign in nature (Note: they can be compromised and used for hosting malicious content so great care needs to be taken here).
    * Below are several data sources for popular domain names.  Each are slightly different in how they measure popularity (by volume of Web visitors, frequency of occurrence in Web Crawling data, by volume of DNS queries based, or a combination).  These lists should not be used as-is for whitelisting; they need to be filtered/refined.  See section on "Building / Maintaining Whitelist Data" below for more details on recommendations for refinement.  
        * [Amazon Alexa top 1 Million](http://s3.amazonaws.com/alexa-static/top-1m.csv.zip)
        * [Cisco Umbrella Top 1 Million](http://s3-us-west-1.amazonaws.com/umbrella-static/index.html)
        * [Domcop Top 10m Domains](https://www.domcop.com/top-10-million-websites) ([data](https://www.domcop.com/files/top/top10milliondomains.csv.zip)) - The top 10 million websites taken from the Open PageRank Initiative.
        * [Majestic Million Domains](https://blog.majestic.com/development/majestic-million-csv-daily/)
        * [Moz's list of the most popular 500 websites on the internet](https://moz.com/top500)
        * [Quantcast Top 1 Million](http://ak.quantcast.com/quantcast-top-sites.zip)
        * [Tranco: A Research-Oriented Top Sites Ranking Hardened Against Manipulation](https://tranco-list.eu/)
    * For more details on these popular domain lists, checkout this [post](/three-short-links-on-popular-domain-lists-for-threat-intelligence/):
* **Popular IP Addresses** - Popular IPs are very similar to popular domains.  They show up everywhere and when they wind up on threat intelligence feeds they cause a lot of false positives.  Popular IP lists can be generated from resolving the Popular domain lists.  These lists should not be used as-is for whitelisting; they need to be filtered/refined.  See section on "Building / Maintaining Whitelist Data" below for more details on recommendations for refinement.  
* **Free email domains** - free email domains occasionally show up in threat intelligence feeds by accident so it is good to maintain a good list of these to prevent false positives.  Below is one source decent source from hubspot.
    * https://knowledge.hubspot.com/articles/kcs_article/forms/what-domains-are-blocked-when-using-the-forms-email-domains-to-block-feature 
* **Ad servers** - Ad servers show up very frequently in URL sandbox feeds as these feeds are often obtained by visiting many websites and waiting for exploitation attempts or for AV alerts.  These same servers show up all the time in benign Internet traffic.  Below is an example data source for these.
    * https://easylist.to/
* **CDN IPs** - Content Distribution Networks are geographically distributed network of proxy servers or caches to help their customers provide high availability and high performance for web content distribution. Their servers are massively shared for distributing varied web content.  When IPs from CDNs make it into threat intelligence feeds, false positives are soon to follow.  Below are several CDN IP and domain patterns.
    * WPO-Foundation CDN list (embedded in Python code) https://github.com/WPO-Foundation/wptagent/blob/master/internal/optimization_checks.py#L62-L286
    * AWS Cloudfront and S3 - https://ip-ranges.amazonaws.com/ip-ranges.json  - filtered for cloudfront and S3 IP space.
    * Cloudflare - https://www.cloudflare.com/ips-v4 
    * Fastly - https://api.fastly.com/public-ip-list 
    * MaxCDN - https://www.maxcdn.com/one/assets/ips.txt 
* **Certificate Revocation Lists (CRL) and the Online Certificate Status Protocol (OCSP) domains/URLs** - When executing a binary in a malware sandbox and the executable has been signed, a lot of connections will be made to CRL and OCSP servers.  Because of this, these often mistakenly wind up in threat feeds.
    * Grab Certificates from Alexa domains, extract OCSP URL.  This old Alienvault post describes the process (along with another approach using the now defunct EFF SSL Observatory): https://cybersecurity.att.com/blogs/labs-research/massively-collecting-crl-and-ocsp-information  and this github repo provides the code to do it https://github.com/pmurgatroyd/alienvault-labs-garage/tree/master/certs.
* **NTP Servers**
    * Web scrape lists of NTP servers (such as https://tf.nist.gov/tf-cgi/servers.cgi and http://www.pool.ntp.org/en/) and perform DNS resolutions to derive all the servers behind each regional load balancer.
* **Root Nameservers and TLD Nameservers**
    * Perform NS lookup against each domain in https://publicsuffix.org/list/public_suffix_list.dat and then perform DNS A-lookup each nameserver domain to obtain their IP addresses.
* **Mail Exchange servers**
    * Obtain a list of popular email domains and then perform MX lookups against popular email domains to get their respective Mail Exchange (MX) servers.   Perform DNS A-lookups on the MX servers list to obtain their IP addresses.
* **STUN Servers** - "Session Traversal Utilities for NAT (STUN) is a standardized set of methods, including a network protocol, for traversal of network address translator (NAT) gateways in applications of real-time voice, video, messaging, and other interactive communications.” via https://en.wikipedia.org/wiki/STUN.  Below are some sources of STUN servers (most of these appear old though).
    * https://www.voip-info.org/stun/ 
    * https://gist.github.com/mondain/b0ec1cf5f60ae726202e 
    * https://gist.github.com/zziuni/3741933 
    * http://enumer.org/public-stun.txt
* **Parking IPs** - IPs used as the default IP for DNS-A records for brand new registered domains.  
    * https://github.com/stamparm/maltrail/blob/master/trails/static/suspicious/parking_site.txt 
* **Popular Open DNS Resolvers**
* **Security Blogs and security tool sites** - These sites show up in threat mailing lists a lot which are sometimes scraped as threat feeds and these domains are mistakenly flagged as malicious.
    * Scrape all the reputable awesome-* security related github repo’s.  This is a little risky since an adversary could potentially get their domain added to these lists.  Examples:
        * https://github.com/sbilly/awesome-security 
        * https://github.com/rshipp/awesome-malware-analysis 
        * https://github.com/paralax/awesome-honeypots 
        * etc.
    * MISP-warninglists provides one that looks pretty good https://github.com/MISP/misp-warninglists/blob/master/lists/security-provider-blogpost/list.json
* **Bit Torrent Trackers** - https://github.com/ngosang/trackerslist 
* **Tracking domains** - commonly used by well known email marketing companies.  Often shows up in threat intel feeds derived from spam or phishing email sinkholes.  Results in high false positive rates in practice.
    * PDNS and/or Domain Whois analytics are one way to identify these once patterns can be observed.  Below is an example of using Whois data for Marketo.com and identifying all the other Marketo email tracking domains that use Marketo’s nameserver.  This example is from https://whoisology.com/ns1/ns1.marketo.com/1

![Marketo Example](/images/whitelists/whoisology.com-ns1.marketo.com.png)

**Note:** MISP-warninglists provides some of these items today https://github.com/MISP/misp-warninglists but they may be stale.  Ideally all of these lists are kept up-to-date through automated collection from authoritative sources.  See section on "Building / Maintaining Whitelist Data” for more tips.

## Benign Host-based Observables

Benign Host-based Observables show up very commonly in threat intelligence feeds based on malware sandboxing.  Here are some example observable types.  So far, I have only found decent benign lists for File hashes (see below).  

* File hashes
* Mutexes
* Registry Keys
* File Paths
* Service names

Data Sources

* https://www.nist.gov/itl/ssd/software-quality-group/nsrl-download/current-rds-hash-sets 
* https://www.nist.gov/itl/ssd/software-quality-group/windows-732-diskprint 
* https://gist.github.com/Neo23x0/fd9af35c5061578025d00838c215dfe4 
* https://github.com/MISP/misp-warninglists/blob/master/lists/common-ioc-false-positive/list.json

## Greylist Observables

Greylist observables are IPs, CIDRs, or domains that we want to ensure never make their way onto the whitelist OR they should influence the score of the domain or IP such that they are rated less trustworthy.  Examples:

* **Shared hosting domains and Dynamic DNS domains** - these base domains should never be alerted on as many are in the Alexa top 1m list and will be incredibly noisy.  BUT subdomains of these are fair game for alerting as they are easily adversary controlled and often abused.
    * **Shared Hosting** - https://github.com/stamparm/maltrail/blob/master/trails/static/suspicious/free_web_hosting.txt 
    * **Dynamic DNS** - https://github.com/stamparm/maltrail/blob/master/trails/static/suspicious/dynamic_domain.txt 
* **DNS Sinkhole IPs**
    * https://tisiphone.net/2017/05/16/consolidated-malware-sinkhole-list/ 
    * https://github.com/brakmic/Sinkholes 

## Building / Maintaining Whitelist Data

Whitelist generation needs to be automated in order to be maintainable.  There may be exceptions to this rule for things that you want to ensure are always in the whitelist, but for everything else, ideally they are collected from authoritative sources or are generated based on sound analytic techniques.  You cannot always blindly trust each data source listed above.  For several, some automated verification, filtering, or analytics will be needed.  Below are some tips for how to do this effectively.

* Each entity in the whitelist should be categorized (what type of whitelist entry is this?) and sourced (where did this come from?) so we know exactly how it got there (i.e. what data source was responsible) and when it was added/updated.  This will help if there is ever a problem related to the whitelist so the specific source of the problem can be addressed.
* Retrieve whitelist entries from the original source sites and parse/extract data from there.  Avoid one time dumps of whitelist entries where possible since these will become stale very quickly.  If you are including one-time dumps be sure to maintain their linage.
* Exclusions to the whitelist should be managed through the greylist.  I.e. if an observable is in the greylist, it should not be evaluated by the whitelist.  
* Several bulk data sets will be very useful for analytics to expand of filter various whitelists
    * **Bulk Active DNS resolution** (A-lookups, MX-lookups, NS-lookups, and TXT-lookups). Adns (https://www.gnu.org/software/adns/) may be useful for this.
    * **Bulk RDNS data** (either from scans.io or collected yourself).
    * **Bulk Whois data** - This can be purchased from several vendors.  Here are a few: https://www.whoisxmlapi.com/, https://iqwhois.com/whois-database-download, https://jsonwhois.com/whois-database-download, https://whoisdatabasedownload.com/, and http://research.domaintools.com/bulk-parsed-whois/
    * **Passive DNS (PDNS) data** - PDNS data can be purchased from several vendors or you can instrument your own network to collect and store this data.  Here are some PDNS suppliers: https://www.farsightsecurity.com/solutions/dnsdb/, https://www.deteque.com/passive-dns/, https://www.circl.lu/services/passive-dns/, https://www.riskiq.com/products/passivetotal/, https://www.coresecurity.com/content/core-pdns
* Netblock ownership (Maxmind) lookups / analytics will be useful for some of the vetting.
* The whitelist should be updated at least daily to stay fresh.  There may be data sources that change more or less frequently than this.  
    * BE CAREFUL when refreshing the whitelist.  Add sanity checks to ensure that the new whitelist was generated correctly before replacing the old one.  The costs of a failed whitelist load will be mass false positives (I learned this lesson the hard way ...).
* Popular domain lists cannot be taken at face value as benign.  Malicious domains get into these lists.  Here are some ways to combat this:
    * Use the **N-day stable top-X technique** - e.g. Stable 6-month Alexa top 500k - create a derivative list from the top Alexa domains where you filter the list for only domains that have been on the Alexa top 500k list every day for the past 6 months.  This technique is commonly used in malicious domain detection literature as a way to build high quality benign labeled data.  It is not perfect and may need to be tuned based on how the whitelist is being used.  This technique requires keeping historic popular domain lists.  The [wayback machine](https://web.archive.org) appears to have a [large historic mirror of the Alexa top1m data](https://web.archive.org/web/20190901000000*/http://s3.amazonaws.com/alexa-static/top-1m.csv.zip) that may be suitable for bootstrapping your own collection.
* Bulk DNS resolution of these lists can also be useful for generating Popular IP lists, but only when using the N-day stable top-X concept or if great care is taken in how they are used.
* Use a greylist for removing categories of domains/IPs that you never want whitelisted.  The greylist should also be kept fresh through automated collection from authoritative sources (e.g. scraping dynamic DNS providers and shared hosting websites where possible, PDNS / Whois analytics may also work).
* Lastly, be careful when generating whitelists and think about what aspects of the data are adversary controlled.  These are things we need to be careful not to blindly trust.  Some examples:
    * RDNS entries can be made to be deceptive esp if the adversary knows they are used for whitelisting.  For these a forward lookup (A Lookup) is generally also needed OR a netblock ownership verification.

As always, feedback is welcome so please leave a message here, on [Medium](https://medium.com/@jason_trost), or @ me on [twitter]((https://twitter.com/#!/jason_trost))!

--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)

