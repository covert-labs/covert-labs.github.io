---
layout: post
title: 6 Short Links on PDNS Graph Analytics for Security
description: "A short listing of recent papers I've read or plan to read using passive DNS data and graph analytics for identifying malicious domains"
modified: 2017-08-08
tags: [security, research, papers, pdns, passive dns]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
---

A short listing of research papers I've read or plan to read that use passive DNS (PDNS) data and graph analytics for identifying malicious domains.

## Host-Domain Graphs

Host domain graphs are bipartite graphs mapping hosts/IPs to domains that they either resolved (passive DNS) or visited (web proxy logs).  These graphs are used heavily in operational security machine learning papers on network threat hunting as they provide insight into the behavioral patterns across an enterprise or ISP.

**[Detecting Malicious Domains via Graph Inference](/research-papers/security/Detecting malicious domains via graph inference.pdf)**
P. K. Manadhata, S. Yadav, P. Rao, and W. Horne.
In Proceedings of 19th European Symposium on Research in Computer Security, Wroclaw, Poland, September 7-11, 2014. 

**[Detection of Early-Stage Enterprise Infection by Mining Large-Scale Log Data](/research-papers/security/Detection of Early-Stage Enterprise Infection by Mining Large-Scale Log Data.pdf)**
Alina Oprea, Zhou Li, Ting-Fang Yen, Sang H. Chin, and Sumyah Alrwais 
In Proceedings of IEEE/IFIP International Conference on Dependable Systems and Networks (DSN), 2015.

**[Segugio: Efficient Behavior-Based Tracking of Malware-Control Domains in Large ISP Networks](/research-papers/security/Segugio - Efficient Behavior-Based Tracking of Malware-Control Domains in Large ISP Networks.pdf)**
Babak Rahbarinia and Manos Antonakakis
In Proceedings of IEEE/IFIP International Conference on Dependable Systems and Networks (DSN), 2015


## Domain Resolution Graphs (Domain-IP Graphs)

A domain resolution graph is an undirected bipartite graph representing observed domain->IP DNS resolution from Passive DNS data.

**[Notos: Building a Dynamic Reputation System for DNS](/research-papers/security/Notos - Building a dynamic reputation system for dns.pdf)**
M. Antonakakis, R. Perdisci, D. Dagon, W. Lee, and N. Feamster. 
In the Proceedings of the 19th USENIX Security Symposium, Washington, DC, USA, August 11-13, 2010.

**[EXPOSURE: Finding Malicious Domains using Passive DNS Analysis](/research-papers/security/Exposure - Finding malicious domains using passive dns analysis.pdf)**
L. Bilge, E. Kirda, C. Kruegel, and M. Balduzzi. 
In Proceedings of the Network and Distributed System Security Symposium, San Diego, California, USA, February 2011.

**[Discovering Malicious Domains through Passive DNS Data Graph Analysis](/research-papers/security/Discovering Malicious Domains through Passive DNS Data Graph Analysis.pdf)**
Issa Khalil, Ting Yu, and Bei Guan. 
In Proceedings of the 11th ACM on Asia Conference on Computer and Communications Security (ASIA CCS '16), 2016.


--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)


*The "short links" format was inspired by [Oreilly's Four Short Links](https://www.oreilly.com/feed/four-short-links) series.*
