---
layout: post
title: Getting Started with DGA Domain Detection Research
description: "Resources for getting started with research on Domain Generation Algorithm (DGA) Domain Detection"
modified: 2020-03-22
tags: [security, research, machine learning, deep learning, DGA]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
---

This post provides resources for getting started with research on Domain Generation Algorithm (DGA) Domain Detection.

DGA Domains are commonly used by malware as a mechanism to maintain a command and control (C2) and make it more difficult for defenders to block. Prior to DGA domains, most malware used a small hardcoded list of IPs or domains. Once these IPs / domains were discovered they could be blocked by defenders or taken down for abuse. DGA domains make this more difficult since the C2 domain changes frequently and enumerating and blocking all generated domains can be expensive.

Recently, I have been working on a [research project](http://www.covert.io/auxiliary-loss-optimization-for-hypothesis-augmentation-for-dga-domain-detection/) recently related to DGA detection (hopefully it will turn into a blogpost or a presentation somewhere), and it occurred to me that DGA is probably one of the most accessible areas for those getting into security data science due to the availability of so much labelled data and the availability of so many open source implementations of DGA detection.  One might argue that this means it is not an area worth researching due to saturation, but I think that depends on your situation/goals.  This short posts outlines some of the resources that I found useful for DGA research.

## Data:

This section lists some domain lists and DGA generators that may be useful for creating "labelled" DGA domain lists.

### DGA Data:

* [DGArchive](https://dgarchive.caad.fkie.fraunhofer.de/) Large private collection of DGA related data.  This contains \~88 csv files of DGA domains organized by malware family.  DGArchive is password protected and if you want access you need to reach out to the maintainer.
* [Bambenek Feeds](https://osint.bambenekconsulting.com/feeds/) (see "DGA Domain Feed").
* [Netlab 360 DGA Feeds](https://data.netlab.360.com/dga/)

### DGA Generators:

* [baderj/domain_generation_algorithms](https://github.com/baderj/domain_generation_algorithms) (276-stars on Github) by [Johannes Bader](https://twitter.com/viql) - DGA algorithms implemented in python.
* [andrewaeva/DGA](https://github.com/andrewaeva/DGA) (123-stars on Github) - smaller collection of DGA algorithms and data, but fills in some of the gaps from domain_generation_algorithms.
* [pchaigno/dga-collection](https://github.com/pchaigno/dga-collection) (37-stars on Github)

#### Word-based / Dictionary-based DGA Resources:

Below are all Malware Families that use word-based / dictionary DGAs, meaning their domains consist of 2 or more words selected from a list/dictionary and concatenated together. I separate these out since they are different than most other "classical" DGAs.

* [Matsnu DGA generator](https://github.com/andrewaeva/DGA/blob/master/dga_algorithms/Matsnu.py)
* [gozi DGA generator](https://github.com/baderj/domain_generation_algorithms/blob/master/gozi/dga.py)
* [suppobox DGA generator](https://github.com/baderj/domain_generation_algorithms/blob/master/suppobox/dga.py)
* [nymaim2 DGA generator](https://github.com/baderj/domain_generation_algorithms/blob/master/nymaim2/dga.py)
* [pizd DGA generator](https://github.com/baderj/domain_generation_algorithms/blob/master/pizd/pizd)

### "Benign" / Non-DGA Data:

This section lists some domain lists that may be useful for creating "labelled" benign domain lists.  In several academic papers one or more of these sources are used, but they generally create derivatives that represent the Stable N-day Top X Sites (e.g. Stable Alexa 30-day top 500k -- meaning domains from the Alexa top 500k that have been on the list consecutively for the last 30 days straight -- the alexa data needs to be downloaded each day for 30+ days to create this since only today's snapshot is provided by Amazon).  This filters out domains that can become popular for a short amount of time but them drop off as sometimes happens with malicious domains.

* [Alexa Top 1M Domains](http://s3.amazonaws.com/alexa-static/top-1m.csv.zip)
* [Quantcast](https://ak.quantcast.com/quantcast-top-sites.zip)
* [Cisco Umbrella Top 1 million](http://s3-us-west-1.amazonaws.com/umbrella-static/top-1m.csv.zip)
* [The Majestic Million](http://downloads.majestic.com/majestic_million.csv)
* [DomCop Top 1M](https://www.domcop.com/files/top/top10milliondomains.csv.zip)
* [Whitelisted Domains](https://raw.githubusercontent.com/maravento/blackweb/master/bwupdate/lst/whiteurls.txt) from [maravento/blackweb](https://github.com/maravento/blackweb)

#### Update (2020-03-22) - More Heuristics for Benign training set curation:

Excerpt from [Inline Detection of DGA Domains Using Side Information (page 12)](https://arxiv.org/pdf/2003.05703.pdf)

> The benign samples are collected based on a predefined set of heuristics as listed below:
> * Domain name should have valid DNS characters only (digits, letters, dot and hyphen)
> * Domain has to be resolved at least once for every day between June 01, 2019 and July 31, 2019.
> * Domain name should have a valid public suffix
> * Characters in the domain name are not all digits (after removing '.' and '-')
> * Domain should have at most four labels (Labels are sequence of characters separated by a dot)
> * Length of the domain name is at most 255 characters
> * Longest label is between 7 and 64 characters
> * Longest label is more than twice the length of the TLD
> * Longest label is more than 70% of the combined length of all labels
> * Excludes IDN (International Distribution Network) domains (such as domains starting with xn--)
> * Domain must not exist in DGArchive


### Utilities:

**Domain Parser:**

When parsing the various domain list data [tldextract](https://pypi.org/project/tldextract/) is very helpful for stripping off TLDs or subdomains if desired.  I have seen several projects attempt to parse domains using "split('.')" or "domain[:-3]".  This does not work very well since domain's TLDs can contain multiple "."s (e.g. .co.uk)

**Installation:**

```
pip install tldextract
```

**Example:**

```
In [1]: import tldextract
In [2]: e = tldextract.extract('abc.www.google.co.uk')

In [3]: e                                                                                                                            Out[3]: ExtractResult(subdomain='abc.www', domain='google', suffix='co.uk')

In [4]: e.domain
Out[4]: 'google'

In [5]: e.subdomain
Out[5]: 'abc.www'

In [6]: e.registered_domain
Out[6]: 'google.co.uk'

In [7]: e.fqdn
Out[7]: 'abc.www.google.co.uk'

In [8]: e.suffix
Out[8]: 'co.uk'
```

**Domain Resolution:**

During the course of your research you may need to perform DNS resolutions on lots of DGA domains.  If you do this, I highly recommend setting up your own bind9 server on Digital Ocean or Amazon and using adnshost (a utility from [adns](https://www.gnu.org/software/adns/)).  If you perform the DNS resolutions from your home or office, your ISP may interfere with the DNS responses because they will appear malicious, which can bias your research.  If you use a provider's recursive nameservers, you may violate the acceptable use policy (AUP) due to the volume AND the provider may also interfere with the responses.  

Adnshost enables high throughput / bulk DNS queries to be performed asynchronously.  It will be much faster than performing the DNS queries synchronously (one after the other).

Here is an example of using adnshost (assuming you are running it from the Bind9 server you setup):

```
cat huge-domains-list.txt | adnshost \
    --asynch \
    --config "nameserver 127.0.0.1" \
    --type a \
    --pipe \
    ----addr-ipv4-only > results.txt
```

This [article](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-14-04) should get you most of the way there with setting up the bind9 server.

## Models:

This section provides links to a few models that could be used as baselines for comparison.

* [dga_predict](https://github.com/endgameinc/dga_predict)'s LSTM and Bigram model from Endgame.
* [snowman](https://github.com/keeganhines/snowman)'s CNN model from [Keegan Hines](https://twitter.com/keeghin).  This is not specifically designed for DGA, but it works for this.
* [matthoffman/degas](https://github.com/matthoffman/degas) - DGA-generated domain detection using deep learning models.
* [#dga-detection](https://github.com/topics/dga-detection), [#dga](https://github.com/topics/dga), and [#dga-domains](https://github.com/topics/dga-domains) on Github - these tags provide other DGA related projects (DGA domain generators, DGA detection, DGA domain lists).
* [BKCS-HUST/LSTM-MI](https://github.com/BKCS-HUST/LSTM-MI)


## Research:

* [EXPOSURE: Finding Malicious Domains Using Passive DNS Analysis](https://sites.cs.ucsb.edu/~chris/research/doc/ndss11_exposure.pdf)
* [From Throw-Away Traffic to Bots: Detecting the Rise of DGA-Based Malware](https://www.usenix.org/system/files/conference/usenixsecurity12/sec12-final127.pdf)
* [Detecting Algorithmically Generated Domain-Flux Attacks With DNS Traffic Analysis](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.221.4391&rep=rep1&type=pdf)
* [Clairvoyant Squirrel: Large-scale malicious domain classification (FloCon 2013)](https://resources.sei.cmu.edu/asset_files/Presentation/2013_017_101_51242.pdf) -- shameless plug :), I worked on this.
* [(DGArchive) A Comprehensive Measurement Study of Domain Generating Malware](https://www.usenix.org/system/files/conference/usenixsecurity16/sec16_paper_plohmann.pdf)
* [DeepDGA: Adversarially-Tuned Domain Generation and Detection](https://arxiv.org/pdf/1610.01969.pdf)
* [A LSTM based Framework for Handling Multiclass Imbalance in DGA Botnet Detection](https://www.researchgate.net/publication/321165269_A_LSTM_based_Framework_for_Handling_Multiclass_Imbalance_in_DGA_Botnet_Detection) [[code]](https://github.com/BKCS-HUST/LSTM-MI)
* [Character Level based Detection of DGA Domain Names](http://faculty.washington.edu/mdecock/papers/byu2018a.pdf) [[code]](https://github.com/matthoffman/degas)
* [Dictionary Extraction and Detection of Algorithmically Generated Domain Names in Passive DNS Traffic](http://faculty.washington.edu/mdecock/papers/mpereira2018a.pdf)
* [An Evaluation of DGA Classifiers](http://faculty.washington.edu/mdecock/papers/rsivaguru2018a.pdf)
* [FANCI : Feature-based Automated NXDomain Classification and Intelligence](https://www.usenix.org/system/files/conference/usenixsecurity18/sec18-schuppen.pdf)
* [Deep Learning For Realtime Malware Detection (ShmooCon 2018)](https://www.youtube.com/watch?v=99hniQYB6VM) by Domenic Puzio and Kate Highnam. -- shameless plug :), I worked with Dom and Kate on several projects.
* [A Novel Detection Method for Word-Based DGA](https://link.springer.com/chapter/10.1007/978-3-030-00009-7_43)
* [A Survey on Malicious Domains Detection through DNS Data Analysis](https://arxiv.org/pdf/1805.08426.pdf)
* [Detecting DGA domains with recurrent neural networks and side information](https://arxiv.org/pdf/1810.02023.pdf)
* [CharBot: A Simple and Effective Method for Evading DGA Classifiers](https://arxiv.org/pdf/1905.01078.pdf)

---

I hope this is helpful.  As always, feedback is welcome so please leave a message here, on [Medium](https://medium.com/@jason_trost), or @ me on [twitter]((https://twitter.com/#!/jason_trost))!

--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)

