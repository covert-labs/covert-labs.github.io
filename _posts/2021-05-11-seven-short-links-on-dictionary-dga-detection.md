---
layout: post
title: "Seven Short Links of Dictionary DGA Detection"
description: ""
modified: 2021-05-11
tags: [security, research, machine learning, DGA, dictionary DGA]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
---

In this short blog, I share seven papers that focus on detecting Dictionary Domain Generation Algorithm (DGA) domains, A.K.A. Word-based DGAs. Dictionary DGAs are algorithms seen in various malware families (suppobox, matsnu, gozi, rovnix, etc.) that are used to periodically generate a large number of domain names that use pseudo-randomly concatenated words from a dictionary.  These domains may appear legitimate at first glance and are often able to evade blacklisting as well as traditional DGA detections based on entropy or counts of consonants vs vowels.  Below are a small sample of rovnix domains from [Unit42's blogpost](https://unit42.paloaltonetworks.com/rovnix-declaration-generation-algorithm/).

* kingwhichtotallyadminis[.]biz
* thareplunjudiciary[.]net
* townsunalienable[.]net
* taxeslawsmockhigh[.]net
* transientperfidythe[.]biz
* inhabitantslaindourmock[.]cn
* thworldthesuffer[.]biz

**Papers:**

* [Real-Time Detection of Dictionary DGA Network Traffic using Deep Learning](https://arxiv.org/pdf/2003.12805.pdf)
* [A Word Graph Approach for Dictionary Detection and Extraction in DGA Domain Names](https://machine-learning-and-security.github.io/slides/Mayana-final-of-NIPS-DDGA.pdf)
* [Dictionary Extraction and Detection of Algorithmically Generated Domain Names in Passive DNS Traffic](http://faculty.washington.edu/mdecock/papers/mpereira2018a.pdf)
* [Inline Detection of Domain Generation Algorithms with Context-Sensitive Word Embeddings](https://arxiv.org/pdf/1811.08705.pdf)
* [An Evaluation of DGA Classifiers](http://faculty.washington.edu/mdecock/papers/rsivaguru2018a.pdf)
* [A Novel Detection Method for Word-Based DGA](https://link.springer.com/chapter/10.1007/978-3-030-00009-7_43)
* [A Word-Level Analytical Approach for Identifying Malicious Domain Names Caused by Dictionary-Based DGA Malware](https://res.mdpi.com/d_attachment/electronics/electronics-10-01039/article_deploy/electronics-10-01039-v2.pdf)

In a previous post, I also shared details on several models that are capable of effectively detecting dictionary DGA domains as well.  Please see [Auxiliary Loss Optimization for Hypothesis Augmentation for DGA Domain Detection](/auxiliary-loss-optimization-for-hypothesis-augmentation-for-dga-domain-detection/).

Lastly, if you're interested in discovering more interesting papers like these, use the method I outlined [here](/security-data-science-learning-resources/).

--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)

The "short links" format was inspired by [O'Reilly's Four Short Links](https://www.oreilly.com/feed/four-short-links) series.
