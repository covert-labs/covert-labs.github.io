---
layout: post
title: "9 Short links on Network Beacon Detection"
description: ""
modified: 2022-01-16
tags: [security, beacons, research, threat hunting, c2]
image:
  feature: beacons-header.png
comments: true
share: true
---

In this post I share 9 links to resources related to Network Beacon detection.  

Network beacons are continuous automated communications between 2 hosts.  Network beacon detection focuses on identifying this automated traffic with the primary goal of aiding in detecting malware infections or adversary activity that have been missed by other controls.  

Beacon detection is a useful building block analytic with many different usecases.

* Threat Hunting and Malware command and control (C2) detection - aid in detecting malware missed by anti-virus products.
* Detection of automated third party traffic - detection of ongoing automated traffic to third parties may reveal unknown or emerging business relationships.
* Identify automated web application dependencies (within an enterprise or external to an enterprise)

Links:

* [Identifying beaconing malware using Elastic](https://www.elastic.co/blog/identifying-beaconing-malware-using-elastic) [[code]](https://github.com/elastic/detection-rules/releases/tag/ML-Beaconing-20211216-1) by Apoorva Joshi, Thomas Veasey, and Craig Chamberlain - uses statistical techniques of coefficient of variation (COV), relative variance (RV), and autocorrelation; implemented as Elastic Painless scripts.
* Enterprise Scale Threat Hunting: C2 Beacon Detection with Unsupervised ML and KQL — [[Part 1](https://posts.bluraven.io/enterprise-scale-threat-hunting-network-beacon-detection-with-unsupervised-machine-learning-and-277c4c30304f)] [[Part 2](https://posts.bluraven.io/enterprise-scale-threat-hunting-network-beacon-detection-with-unsupervised-ml-and-kql-part-2-bff46cfc1e7e)] [[code]](https://github.com/Cyb3r-Monk/Threat-Hunting-and-Detection/tree/main/Command%20and%20Control) by Mehmet Ergene
* [Detecting network beacons via KQL using simple spread stats functions](https://ateixei.medium.com/detecting-network-beacons-via-kql-using-simple-spread-stats-functions-c2f031b0736b) by Alex Teixeira
* [Detect Network beaconing via Intra-Request time delta patterns in Azure Sentinel](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/detect-network-beaconing-via-intra-request-time-delta-patterns/ba-p/779586) [[code]](https://github.com/Azure/Azure-Sentinel/blob/master/Detections/CommonSecurityLog/PaloAlto-NetworkBeaconing.yaml) by Ashwin Patil
* [RITA (Real Intelligence Threat Analytics) beacon analyzer](https://github.com/activecm/rita/blob/master/pkg/beacon/analyzer.go) - uses simple statistical approach based on 6 measures: connection time delta skew, connection dispersion, connection counts over time, data size skew, data size dispersion, and data size smallness score.
* [How to detect beaconing traffic with Splunk?](https://github.com/inodee/threathunting-spl/blob/master/hunt-queries/Detecting_Beaconing.md) by Alex Teixeira
* [Detect Beaconing with Flare, Elastic Stack, and Intrusion Detection Systems](http://www.austintaylor.io/detect/beaconing/intrusion/detection/system/command/control/flare/elastic/stack/2017/06/10/detect-beaconing-with-flare-elasticsearch-and-intrusion-detection-systems/) [[code]](https://github.com/austin-taylor/flare/blob/master/flare/analytics/command_control.py) by Austin Taylor
* [BAYWATCH: Robust Beaconing Detection to Identify Infected Hosts in Large-Scale Enterprise Networks](https://alps-lab.github.io/paper/hu-dsn-2016.pdf) - uses FFT and periodogram based technique for identifying automated traffic.
* [Malware Beaconing Detection by Mining Large-scale DNS Logs for Targeted Attack Identification](https://publications.waset.org/10004242/malware-beaconing-detection-by-mining-large-scale-dns-logs-for-targeted-attack-identification)

--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)

The "short links" format was inspired by [O'Reilly's Four Short Links](https://www.oreilly.com/feed/four-short-links) series.
