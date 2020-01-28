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

Whitelisting is a useful concept in Threat Intelligence correlation since it can be very easy for benign observables to make their way into [threat intelligence indicator feeds](/threat-intelligence/), esp. coming from open source providers or vendors that are not as careful as they should be.  If these threat intelligence feeds are used for blocking (e.g. in firewalls or WAF devices) or alerting (e.g. log correlation in SIEM or IDS), the cost of benign entries making their way into a security control will be very high (wasted analyst time for triaging false positive alerts or loss of business productivity for blocked legitimate websites).  Whitelists are generally used to filter out observables from threat intelligence feeds that almost certainly would be marked as a false positive if they were intersected against event logs (e.g. bluecoat proxy logs, firewall logs, etc) and used for alerting.  Whitelists are also very useful for building labeled datasets required for building machine learning models and enriching alerts with contextual information.

The classic example of a benign obserable is 8.8.8.8 (Google's published open DNS resolver).  This has found its way into many open source and commercial threat intelligence feeds by mistake since sometimes malware use this IP for DNS resolution or they ping it for connectivity checks.  There are many other observables that commonly make their way into threat feeds due to how the threat feeds are derived / collected.  Below are a summary of the major sources of false positives for threat intelligence feeds and ways to identify these to prevent their use.  Most commercial threat intelligence platforms are pretty good at identifying these today and the dominant open source threat intelligence platform MISP is getting better with its [MISP-warninglists](https://github.com/MISP/misp-warninglists/), but as you will discover below there is some room for improvement.

## Benign Inbound Observables

Benign Inbound Observables commonly show up in threat intelligence feeds derived from distributed network sensors such as honeypots or firewall logs.  These IPs show up in firewall logs and are generally benign or at best are considered noise.  Below are several common Benign Inbound Observable types. Each type also comes with recommended data sources or collection techniques listed as sub bullets:

* **Known Web Crawlers** - Web crawlers are servers that crawl the World Wide Web and through this process may enter the networks of many companies or may accidentally hit honeypots or firewalls.
    * RDNS + DNS analytics can be used to enumerate these in bulk once patterns are identified.  [Here](https://support.google.com/webmasters/answer/80553?hl=en) is an example pattern for googlebots.  Mining large collections of rdns data can reveal other patterns to focus on.  Below is an example of a simple PTR lookup on a known googlebot IP.  This should start to reveal patterns that could be codified assuming you have access to a large corpus of RDNS data like is provided [here](https://opendata.rapid7.com/sonar.rdns_v2/) (or could easily be generated).  

![Googlebot DNS](/images/whitelists/googlebot-dns.png)

* **Known port scanners** associated with highly visible projects or security companies (Shodan, Censys, Rapid7 Project Sonar, ShadowServer, etc.)
    * RDNS + DNS analytics may be able to enumerate these in bulk (assuming the vendors want to be identified).  Example:

![Shodan DNS](/images/whitelists/shodan-dns.png)

* **Mail Servers** - these servers send email and they sometimes wind up on Threat feeds by mistake.  
    * In order to enumerate these, you need a good list of popular email domains.  Then perform DNS TXT request against this list and parse the SPF records.  Multiple lookups will likely be needed as SPF allows for redirects and includes.  Below shows the commands needed to do this manually for gmail.com as an example.  The CIDR blocks returned are the IP space where gmail emails are sent from.  Alerting or blocking on these is gonna cause a bad day.

![Gmail DNS](/images/whitelists/gmail-dns.png)

* **Cloud PaaS Providers** – Most Cloud providers publish their IP space via APIs or in their documentation.  These lists are useful to derive whitelists, but they will need to be further filtered.  Ideally you only whitelist Cloud IP space that are massively shared (like S3, CLOUDFRONT, etc), not IPs that are easy for bad guys to such as like EC2s.  These whitelists should not be used to exclude domain names that resolve to this IP space, but instead should be used for either enrichments on alerting or to suppress IOC based alerting from these IP ranges.
    * [Amazon AWS IP Ranges](https://ip-ranges.amazonaws.com/ip-ranges.json)
    * [Google Cloud Platform IP Ranges](https://gist.github.com/n0531m/f3714f6ad6ef738a3b0a)
    * [Azure IP Ranges](https://www.microsoft.com/en-us/download/details.aspx?id=56519)

**Note**: [Greynoise](https://greynoise.io) is commercial provider of "anti-threat" intelligence (i.e. they identify the noise and other benign observables).  They are very good at identifying the types of benign observables listed above since they maintain a globally distributed sensor array and are specifically analyzing network events in order to identify benign activity.

**Note**: [MISP-warninglists](https://github.com/MISP/misp-warninglists) provides many of these items today but they may be stale (several of their lists have not been updated in months).  Ideally all of these lists are kept up-to-date through automated collection from authoritative sources instead of hard coded data stored in github (unless these are automatically updated frequently).  See section on "[Building / Maintaining Whitelist Data](#building-maintaining-whitelists)" for more tips.

## Benign Outbound Observables

Benign Outbound Observables show up frequently in threat intelligence feeds derived from malware sandboxing, URL sandboxing, outbound web crawling, email sandboxing, and other similar threat feeds.   Below are several common Benign Outbound Observable types. Each type also comes with recommended data sources or collection techniques listed as sub bullets:

* **Popular Domains** - Popular domains can wind up on threat intelligence feeds, especially those derived from malware sandboxing since often times malware uses benign domains as connectivity checks and some malware, like those conducting click fraud act more like web crawlers, visiting many different benign sites.  These same popular domains show up very often in most corporate networks and are almost always benign in nature (Note: they can be compromised and used for hosting malicious content so great care needs to be taken here).
    * Below are several data sources for popular domain names.  Each are slightly different in how they measure popularity (by volume of Web visitors, frequency of occurrence in Web Crawling data, by volume of DNS queries based, or a combination).  These lists should not be used as-is for whitelisting; they need to be filtered/refined.  See section on "[Building / Maintaining Whitelist Data](#building-maintaining-whitelists)" below for more details on recommendations for refinement.  
        * [Amazon Alexa top 1 Million](http://s3.amazonaws.com/alexa-static/top-1m.csv.zip)
        * [Cisco Umbrella Top 1 Million](http://s3-us-west-1.amazonaws.com/umbrella-static/index.html)
        * [Domcop Top 10m Domains](https://www.domcop.com/top-10-million-websites) ([data](https://www.domcop.com/files/top/top10milliondomains.csv.zip)) - The top 10 million websites taken from the Open PageRank Initiative.
        * [Majestic Million Domains](https://blog.majestic.com/development/majestic-million-csv-daily/)
        * [Moz's list of the most popular 500 websites on the internet](https://moz.com/top500)
        * [Quantcast Top 1 Million](http://ak.quantcast.com/quantcast-top-sites.zip)
        * [Tranco: A Research-Oriented Top Sites Ranking Hardened Against Manipulation](https://tranco-list.eu/)
        * MISP warning-lists' [dax30 websites](https://github.com/MISP/misp-warninglists/tree/master/lists/dax30), [bank websites](https://github.com/MISP/misp-warninglists/tree/master/lists/bank-website), [university domains](https://github.com/MISP/misp-warninglists/tree/master/lists/university_domains), [url shorteners](https://github.com/MISP/misp-warninglists/tree/master/lists/url-shortener), [whats-my-ip sites](https://github.com/MISP/misp-warninglists/tree/master/lists/whats-my-ip)
    * For more details and analysis on these popular domain lists, checkout this [post](/three-short-links-on-popular-domain-lists-for-threat-intelligence/):
* **Popular IP Addresses** - Popular IPs are very similar to popular domains.  They show up everywhere and when they wind up on threat intelligence feeds they cause a lot of false positives.  Popular IP lists can be generated from resolving the Popular domain lists.  These lists should not be used as-is for whitelisting; they need to be filtered/refined.  See section on "[Building / Maintaining Whitelist Data](#building-maintaining-whitelists)" below for more details on recommendations for refinement.  
* **Free email domains** - free email domains occasionally show up in threat intelligence feeds by accident so it is good to maintain a good list of these to prevent false positives.  Hubspot provides a [list](https://knowledge.hubspot.com/articles/kcs_article/forms/what-domains-are-blocked-when-using-the-forms-email-domains-to-block-feature) that is decent.
* **Ad servers** - Ad servers show up very frequently in URL sandbox feeds as these feeds are often obtained by visiting many websites and waiting for exploitation attempts or for AV alerts.  These same servers show up all the time in benign Internet traffic.  [Easylist](https://easylist.to/) provides this sort of data.
* **CDN IPs** - Content Distribution Networks are geographically distributed network of proxy servers or caches that provide high availability and high performance for web content distribution. Their servers are massively shared for distributing varied web content.  When IPs from CDNs make it into threat intelligence feeds, false positives are soon to follow.  Below are several CDN IP and domain sources.
    * [WPO-Foundation CDN list (embedded in Python code)](https://github.com/WPO-Foundation/wptagent/blob/master/internal/optimization_checks.py#L62-L286)
    * [AWS IP Ranges](https://ip-ranges.amazonaws.com/ip-ranges.json) - but filtered for cloudfront and S3 IP space.
    * [Cloudflare IP Ranges](https://www.cloudflare.com/ips-v4)
    * [Fastly IP Ranges](https://api.fastly.com/public-ip-list)
    * [MaxCDN IP Ranges](https://www.maxcdn.com/one/assets/ips.txt)
    * Very similar to identifying known web crawlers, DNS PTR-Lookup + DNS A-Lookup analytics can be used to enumerate these in bulk once patterns are identified.
* **Certificate Revocation Lists (CRL) and the Online Certificate Status Protocol (OCSP) domains/URLs** - When executing a binary in a malware sandbox and the executable has been signed, connections will be made to CRL and OCSP servers.  Because of this, these often mistakenly wind up in threat feeds.
    * Grab Certificates from Alexa top websites, extract OCSP URL.  This [old Alienvault post](https://cybersecurity.att.com/blogs/labs-research/massively-collecting-crl-and-ocsp-information) describes the process (along with another approach using the now defunct EFF SSL Observatory), and this [github repo](https://github.com/pmurgatroyd/alienvault-labs-garage/tree/master/certs) provides the code to do it.  Care should be taken here since adversaries can influence the data collected in this way.
    * [MISP warning-lists crl-ip-hostname](https://github.com/MISP/misp-warninglists/tree/master/lists/crl-ip-hostname)
* **NTP Servers** - Some malware call out to NTP servers for connectivity checks or to determine the real date/time.  Because of this, NTP servers often wind up mistakenly on threat intelligence feeds that are derived from malware sandboxing.  
    * Web scrape lists of NTP servers (such as the [NIST Internet Time Servers](https://tf.nist.gov/tf-cgi/servers.cgi) and [NTP Pool Project Servers](http://www.pool.ntp.org/en/)) and perform DNS resolutions to derive all the servers behind each regional load balancer.
* **Root Nameservers and TLD Nameservers**
    * Perform DNS NS-lookups against each domain in the [Public Suffix List](https://publicsuffix.org/list/public_suffix_list.dat) and then perform DNS A-lookup each nameserver domain to obtain their IP addresses.
* **Mail Exchange servers**
    * Obtain a list of popular email domains and then perform MX lookups against popular email domains to get their respective Mail Exchange (MX) servers.   Perform DNS A-lookups on the MX servers list to obtain their IP addresses.
* **STUN Servers** - "Session Traversal Utilities for NAT (STUN) is a standardized set of methods, including a network protocol, for traversal of network address translator (NAT) gateways in applications of real-time voice, video, messaging, and other interactive communications." via https://en.wikipedia.org/wiki/STUN.  Below are some sources of STUN servers (some of these appear old though).
    * [https://www.voip-info.org/stun/](https://www.voip-info.org/stun)/ 
    * [https://gist.github.com/mondain/b0ec1cf5f60ae726202e](https://gist.github.com/mondain/b0ec1cf5f60ae726202e) 
    * [https://gist.github.com/zziuni/3741933](https://gist.github.com/zziuni/3741933) 
    * [http://enumer.org/public-stun.txt](http://enumer.org/public-stun.txt)
* **Parking IPs** - IPs used as the default IP for DNS-A records for brand new registered domains.  
    * [maltrails parking_sites](https://github.com/stamparm/maltrail/blob/master/trails/static/suspicious/parking_site.txt) 
* **Popular Open DNS Resolvers**
    * [Public recursive name server (Wikipedia)](https://en.wikipedia.org/wiki/Public_recursive_name_server) - lists the largest and most popular open recursive nameservers.
    * [Public DNS Server List](https://public-dns.info/) - maintains a large list of open recursive nameservers that may be useful for context, but should not be whitelisted.
* **Security Companies, Security Blogs and Security Tool sites** - These sites show up in threat mailing lists frequently which are sometimes scraped as threat feeds and these domains are mistakenly flagged as malicious.
    * Scrape all the reputable awesome-* security related github repo’s.  This is a little risky since an adversary could potentially get their domain added to these lists.  Examples:
        * [awesome-security](https://github.com/sbilly/awesome-security)
        * [awesome-malware-analysis](https://github.com/rshipp/awesome-malware-analysis)
        * [awesome-honeypots](https://github.com/paralax/awesome-honeypots)
        * etc.
    * MISP-warninglists provides a [security-provider-blogpost](https://github.com/MISP/misp-warninglists/blob/master/lists/security-provider-blogpost/) and [automated-malware-analysis](https://github.com/MISP/misp-warninglists/tree/master/lists/automated-malware-analysis) lists that look pretty good.
* **Bit Torrent Trackers** - [github.com/ngosang/trackerslist](https://github.com/ngosang/trackerslist) 
* **Tracking domains** - commonly used by well known email marketing companies.  Often shows up in threat intel feeds derived from spam or phishing email sinkholes.  Results in high false positive rates in practice.
    * PDNS and/or Domain Whois analytics are one way to identify these once patterns can be observed.  Below is an example of using Whois data for Marketo.com and identifying all the other Marketo email tracking domains that use Marketo’s nameserver.  This example is from [Whoisology](https://whoisology.com/ns1/ns1.marketo.com/1), but bulk Whois mining is a preferred method.

![Marketo Example](/images/whitelists/whoisology.com-ns1.marketo.com.png)

**Note:** [MISP-warninglists](https://github.com/MISP/misp-warninglists) provides some of these items today but they may be stale.  Ideally all of these lists are kept up-to-date through automated collection from authoritative sources.  See section on "[Building / Maintaining Whitelist Data](#building-maintaining-whitelists)" for more tips.

## Benign Host-based Observables

Benign Host-based Observables show up very commonly in threat intelligence feeds based on malware sandboxing.  Here are some example observable types.  So far, I have only found decent benign lists for File hashes (see below).  

* File hashes
* Mutexes
* Registry Keys
* File Paths
* Service names

Data Sources:

* [NSRL Hashsets](https://www.nist.gov/itl/ssd/software-quality-group/national-software-reference-library-nsrl/nsrl-download)
* [Windows-7/32 Diskprint](https://www.nist.gov/itl/ssd/software-quality-group/windows-732-diskprint) 
* [Neo23x0/fp-hashes.py](https://gist.github.com/Neo23x0/fd9af35c5061578025d00838c215dfe4) 
* [MISP common IOC false positives](https://github.com/MISP/misp-warninglists/blob/master/lists/common-ioc-false-positive/list.json)
* [Mandiant Redline Whitelist (mirror)](https://github.com/kost/m-whitelist) - NOTE: this is \~5yr old at the time of this blog.
* [Hashsets.com (commercial) hash lists](https://www.hashsets.com/white-hash-sets-2/)

In leading academic and industry research on malware detection, it is common to use variations of the following techniques (based on [Virustotal](https://www.virustotal.com/) determinations) in order to build labeled training data.  These techniques seem very suitable for training data creation, but are not recommended for whitelisting for operational use due to the high likelihood of false negatives.

* "In this paper, we use a '1-/5+ criterion for labeling a given file as malicious or benign: if a file has one or fewer vendors reporting it as malicious, we label the file as 'benign'".  See [ALOHA: Auxiliary Loss Optimization for Hypothesis Augmentation](https://www.usenix.org/system/files/sec19-rudd.pdf) for more details.
* "We assign malicious/benign labels on a 5+/1- basis, i.e., documents for which one or fewer vendors labeled malicious, we ascribe the aggregate label benign, while documents for which 5 or more vendors labeled malicious, we ascribe the aggregate label malicious."  See [MEADE: Towards a Malicious Email Attachment Detection Engine](https://arxiv.org/pdf/1804.08162.pdf) for more details.
* Uses similar method as above, but further removes files that use hash based file names or filenames that are "malware" or "sample".  See [Learning from Context: Exploiting and Interpreting File Path Information for Better Malware Detection](https://arxiv.org/pdf/1905.06987.pdf) for more details.
* "To train and evaluate our model at low false positive rates, we require accurate labels for our malware and benignware binaries. We accomplish this by running all of our data through VirusTotal, which runs the binaries through approximately 55 malware engines.We then use a voting strategy to decide if each file is either malware or benignware...  We label any file against which 30% or more of the anti-virus engines alarm as malware, and any file that no anti-virus engine alarms on as benignware. For the purposes of both training and accuracy evaluation we discard any files that more than 0% and less than 30% of VirusTotal's anti-virus engines declare it malware, given the uncertainty surrounding the nature of these files."
  See [Deep Neural Network Based Malware Detection Using Two Dimensional Binary Program Features](https://arxiv.org/pdf/1508.03096.pdf) for more details.
* Scraping packages from [Ninite](https://ninite.com/), [Chocolatey](https://chocolatey.org/), and [Cygwin](https://www.cygwin.com/).


**Note:** If your goal is building a machine learning model on binaries, you should strongly consider Endgame's [Ember](https://github.com/endgameinc/ember).  "The dataset includes features extracted from 1.1M binary files: 900K training samples (300K malicious, 300K benign, 300K unlabeled) and 200K test samples (100K malicious, 100K benign)".  See [EMBER: An Open Dataset for Training Static PE Malware Machine Learning Models](https://arxiv.org/abs/1804.04637) for more details.

## Whitelist Exclusions

There are many observables that we will never want to whitelist due to their popularity or importance.  These should be maintained in a whitelist exclusions list (a.k.a. greylist).  Below are some examples:

* **Shared hosting domains and Dynamic DNS domains** - these base domains should never be alerted on as many are in the Alexa top 1m list and will be incredibly noisy.  BUT subdomains of these are fair game for alerting as they are easily adversary controlled and often abused.  Below are some sources of this information, but identifying the major providers and scraping their websites or APIs would be a better way to keep these fresh.
    * **Shared Hosting** - [Maltrails free web hosting](https://github.com/stamparm/maltrail/blob/master/trails/static/suspicious/free_web_hosting.txt) 
    * **Dynamic DNS** - [Maltrails DynDNS](https://github.com/stamparm/maltrail/blob/master/trails/static/suspicious/dynamic_domain.txt) 
* **DNS Sinkhole IPs**
    * [https://tisiphone.net/2017/05/16/consolidated-malware-sinkhole-list/](https://tisiphone.net/2017/05/16/consolidated-malware-sinkhole-list/)
    * [github.com/brakmic/Sinkholes](https://github.com/brakmic/Sinkholes)
    * [sinkdb.abuse.ch](https://sinkdb.abuse.ch/)
    * [MISP warninglists sinkholes](https://github.com/MISP/misp-warninglists/blob/master/lists/sinkholes/)

## <a name="building-maintaining-whitelists">Building / Maintaining Whitelist Data</a>

Whitelist generation needs to be automated in order to be maintainable.  There may be exceptions to this rule for things that you want to ensure are always in the whitelist, but for everything else, ideally they are collected from authoritative sources or are generated based on sound analytic techniques.  You cannot always blindly trust each data source listed above.  For several, some automated verification, filtering, or analytics will be needed.  Below are some tips for how to do this effectively.

* Each entity in the whitelist should be categorized (what type of whitelist entry is this?) and sourced (where did this come from?) so we know exactly how it got there (i.e. what data source was responsible) and when it was added/updated.  This will help if there is ever a problem related to the whitelist so the specific source of the problem can be addressed.
* Retrieve whitelist entries from the original source sites and parse/extract data from there.  Avoid one time dumps of whitelist entries where possible since these will become stale very quickly.  If you are including one-time dumps be sure to maintain their lineage.
* Several bulk data sets will be very useful for analytics to expand or filter various whitelists
    * **Bulk Active DNS resolution** (A-lookups, MX-lookups, NS-lookups, and TXT-lookups). [Adns](https://www.gnu.org/software/adns/) may be useful for this.
    * **Bulk RDNS data** (either from [scans.io](https://scans.io) or collected yourself).
    * **Bulk Whois data** - This can be purchased from several vendors.  Here are a few: [whoisxmlapi.com](https://www.whoisxmlapi.com/), [iqwhois.com](https://iqwhois.com/whois-database-download), [jsonwhois.com](https://jsonwhois.com/whois-database-download), [whoisdatabasedownload.com](https://whoisdatabasedownload.com/), and [research.domaintools.com](http://research.domaintools.com/bulk-parsed-whois/).
    * **Passive DNS (PDNS) data** - PDNS data can be purchased from several vendors or you can instrument your own network to collect and store this data.  Here are some PDNS suppliers: [farsightsecurity.com](https://www.farsightsecurity.com/solutions/dnsdb/), [deteque.com](https://www.deteque.com/passive-dns/), [circl.lu](https://www.circl.lu/services/passive-dns/), [riskiq.com](https://www.riskiq.com/products/passivetotal/), [passivedns.mnemonic.no](https://passivedns.mnemonic.no/), and [coresecurity.com (formerly Damballa)](https://www.coresecurity.com/content/core-pdns).
* Netblock ownership (Maxmind) lookups / analytics will be useful for some of the vetting.
* The whitelist should be updated at least daily to stay fresh.  There may be data sources that change more or less frequently than this.  
    * BE CAREFUL when refreshing the whitelist.  Add sanity checks to ensure that the new whitelist was generated correctly before replacing the old one.  The costs of a failed whitelist load will be mass false positives (unfortunately, I had to learn this lesson the hard way ...).
* Popular domain lists cannot be taken at face value as benign.  Malicious domains get into these lists all the time.  Here are some ways to combat this:
    * Use the **N-day stable top-X technique** - e.g. Stable 6-month Alexa top 500k - create a derivative list from the top Alexa domains where you filter the list for only domains that have been on the Alexa top 500k list every day for the past 6 months.  This technique is commonly used in malicious domain detection literature as a way to build high quality benign labeled data.  It is not perfect and may need to be tuned based on how the whitelist is being used.  This technique requires keeping historic popular domain lists.  The [Wayback Machine](https://web.archive.org) appears to have a [large historic mirror of the Alexa top1m data](https://web.archive.org/web/*/http://s3.amazonaws.com/alexa-static/top-1m.csv.zip) that may be suitable for bootstrapping your own collection.
* Bulk DNS resolution of these lists can also be useful for generating Popular IP lists, but only when using the N-day stable top-X concept or if great care is taken in how they are used.
* Use a whitelist exclusions set for removing categories of domains/IPs that you never want whitelisted.  The whitelist exclusions set should also be kept fresh through automated collection from authoritative sources (e.g. scraping dynamic DNS providers and shared hosting websites where possible, PDNS / Whois analytics may also work).
* Lastly, be careful when generating whitelists and think about what aspects of the data are adversary controlled.  These are things we need to be careful not to blindly trust.  Some examples:
    * RDNS entries can be made to be deceptive especially if the adversary knows they are used for whitelisting.  For example, an adversary can create PTR records for IP address space they own that are identical to Google's googlebot RDNS or Shodan's census RDNS, BUT they cannot change the DNS A record mapping that domain name back to their IP space.  For these a forward lookup (A Lookup) is generally also needed OR a netblock ownership verification.  

In conclusion, whitelists are useful for filtering out observables from threat intelligence lists before correlation with event data, building labeled datasets for machine learning models, and enriching threat intelligence or alerts with contextual information.  Creating and maintaining these lists can be a lot of work.  Great care should be taken as to not go too far or to whitelist domains or IPs that are easily adversary controlled.

As always, feedback is welcome so please leave a message here, on [Medium](https://medium.com/@jason_trost), or @ me on [twitter]((https://twitter.com/#!/jason_trost))!

--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)

