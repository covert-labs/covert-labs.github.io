---
layout: page
permalink: /data-links/
title: Data
description: "Useful Data for Security Researchers"
tags: [data, security, dns, malware, threat intelligence, iocs, passwords]
image:
  feature: grey-scale-dark.jpg
share: true
---

## Data

Collection of Security and Network Data Resources.  See the [Threat Intelligence](/threat-intelligence/) page for a massive list of threat intelligence feeds.

### Popular Websites / Domains Data

* [Alexa Top 1m Domains](http://s3.amazonaws.com/alexa-static/top-1m.csv.zip)
* [Large historic mirror of the Alexa Top 1m Domains](https://web.archive.org/web/*/http://s3.amazonaws.com/alexa-static/top-1m.csv.zip)
* [Cisco Umbrella Top 1 Million Domains](https://blog.opendns.com/2016/12/14/cisco-umbrella-1-million/) ([data](http://s3-us-west-1.amazonaws.com/umbrella-static/top-1m.csv.zip))
* [Domcop Top 10m Domains](https://www.domcop.com/top-10-million-websites) ([data](https://www.domcop.com/files/top/top10milliondomains.csv.zip)) - The top 10 million websites taken from the Open PageRank Initiative.
* [Majestic Million](https://majestic.com/reports/majestic-million) ([data](http://downloads.majestic.com/majestic_million.csv)) - Top domains list based on web crawling data.
* [Quantcast top Million Domains](https://ak.quantcast.com/quantcast-top-sites.zip)
* [Tranco: A Research-Oriented Top Sites Ranking Hardened Against Manipulation](https://tranco-list.eu/) ([data](https://tranco-list.eu/top-1m.csv.zip)) - another top domains list derived from Alexa, Umbrella, Majestic, and Quantcast, but combined using the Dowdall rule.
* [OpenIntel Historic active DNS data](https://data.openintel.nl/data/) - DNS resolution data for Alexa top1m, Umbrella top1m, and open TLDs (.se, .nu, .ee ccTLDs and all US Federal domain names from the .gov and .fed.us).
* [Adservers](https://github.com/jmdugan/blocklists/tree/master/ad_servers)
* [Tech Company Domains](https://github.com/jmdugan/blocklists/tree/master/corporations)

### Malware

* [Blue Hexagon Open Dataset for Malware AnalysiS (BODMAS)](https://whyisyoung.github.io/BODMAS/)
* [EMBER](https://github.com/elastic/ember) - Endgame Malware BEnchmark for Research
* [Malware Training Sets: A machine learning dataset for everyone](http://marcoramilli.blogspot.cz/2016/12/malware-training-sets-machine-learning.html) ([data](https://github.com/marcoramilli/MalwareTrainingSets))
* [Malware Open-source Threat Intelligence Family (MOTIF)](https://github.com/boozallen/MOTIF)
* [SoReL-20M](https://github.com/sophos-ai/SOREL-20M) - Sophos-ReversingLabs 20 Million dataset.
* [Virusshare](https://virusshare.com/)

### Spyware IOCs

These were found via [this](https://github.com/SpyGuard/SpyGuard/blob/master/watchers.yaml).

* [spyguard spyware IOCs](https://raw.githubusercontent.com/SpyGuard/spyguard/master/assets/iocs.json)
* [spyguard whitelist](https://raw.githubusercontent.com/SpyGuard/spyguard/master/assets/whitelist.json)
* [stalkerware spyware IOCs](https://raw.githubusercontent.com/AssoEchap/stalkerware-indicators/master/generated/indicators-for-tinycheck.json)

### Other Data

* [betterdefaultpasslist](https://github.com/govolution/betterdefaultpasslist) - "list includes default credentials from various manufacturers for their products like NAS, ERP, ICS etc., that are used for standard products like mssql, vnc, oracle and so on"
* [Common Crawl](http://commoncrawl.org/) - TBs of publicly available web crawl data hosted in Amazon S3.
* [Covert.io Threat Intelligence List](/threat-intelligence/)
* [Dark Web Market Crawls with onionscan](https://polecat.mascherari.press/onionscan/dark-web-data-dumps)
* [Darknet Market Archives (2013-2015)](https://www.gwern.net/Black-market%20archives)
* [DARPA Intrusion Detection Data Sets](https://www.ll.mit.edu/ideval/data/)
* [Data Capture from National Security Agency at CDX](http://www.westpoint.edu/crc/SitePages/DataSets.aspx)
* [Data Driven Security Dataset Collection](http://datadrivensecurity.info/blog/pages/dds-dataset-collection.html)
* [DGArchive](https://dgarchive.caad.fkie.fraunhofer.de/site/) - DGA Domains Database.
* [Domains-index.com](https://domains-index.com/) - ccTLD Zone Files / Domain Name Lists for sale.  See [Free Domain Lists](https://domains-index.com/downloads/category/free/)
* [heralding honeypot log sample (3077 events)](/data/heralding_activity.log.gz)
* [Malicious URLs Data Sets](http://sysnet.ucsd.edu/projects/url/)
* [Multi-Source Cyber-Security Events](http://csr.lanl.gov/data/cyber1/)
* [NSL-KDD Data Sets](https://github.com/defcom17/NSL_KDD)
* [Onionscan data sample](https://github.com/automatingosint/osint_public/tree/master/onionrunner)
* [Open Data Sets](http://csr.lanl.gov/data/)
* [PTRarchive](http://ptrarchive.com/) - Massive collection of DNS PTR records data available for search.
* [PublicDB.host](https://publicdb.host/index.php) ([data](https://cdn.publicdb.host/)) - collection of dumped databases from many major breaches.
* [Scans.io](https://scans.io/) - publicly available Internet scale port scan and DNS data.
* [SecRepo.com - Samples of Security Related Data](http://www.secrepo.com/)
* [Stratosphere IPS Data Sets](https://stratosphereips.org/category/dataset.html)
* [The ADFA Intrusion Detection Data Sets](https://www.unsw.adfa.edu.au/australian-centre-for-cyber-security/cybersecurity/ADFA-IDS-Datasets/)
* [VERIS Community Database](https://github.com/vz-risk/VCDB)
* [ViewDns.info](http://viewdns.info/data/) - ccTLD Zone Files / Domain Name Lists for sale.
* [VX Heaven](http://vxheaven.org/faq.php)
* [Web Data Commons (Common Crawl derivatives)](http://webdatacommons.org/) - Extracting Structured Data from the Common Crawl
* [WhoisXML Domain Registration Feeds (Commercial)](https://www.whoisxmlapi.com):
    * [Whois Database Download](https://www.whoisxmlapi.com/whois-database-download.php)
    * [Newly Registered & Just Expired Domains](https://www.whoisxmlapi.com/newly-registered-domains.php)
