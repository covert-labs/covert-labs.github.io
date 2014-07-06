---
layout: post
title: Large Scale Malicious Domain Classification with Storm, Random Forrests, and Markov Models
description: "Large Scale Malicious Domain Classification with Storm, Random Forrests, and Markov Models"
modified: 2013-01-31
tags: [security, machine learning, data science, dns, storm, endgame, bigdata, realtime, random forrest, classifier, markov model]
image:
  feature: grey-scale-dark.jpg
comments: true
share: true
permalink: /post/41859934400/large-scale-malicious-domain-classification-with-storm
---

<center>
<img src="/images/clairvoyant-squirrel.png" height="250" width="250" />
</center>

<br /><br />
At <a href="http://endgame.com/">Endgame</a> we have been working on a system for large scale malicious DNS detection, and <a href="http://www.cert.org/flocon/speakers.html#trost">Myself</a> and <a href="http://www.cert.org/flocon/speakers.html#munro">John Munro</a> recently presented some of this work at <a href="http://www.cert.org/flocon/">FloCon</a>.  
<br /><br />
<b>Abstract:</b>
<blockquote>
<div><b>Clairvoyant Squirrel: Large Scale Malicious Domain Classification</b>
<br /><br />
Large scale classification of domain names has many applications in network monitoring, intrusion detection, and forensics.  The goal with this research is to predict a domain’s maliciousness solely based on the domain string itself, and to perform this classification on domains seen in real-time on high traffic networks, giving network administrators insight into possible intrusions.  Our classification model uses the <a href="http://en.wikipedia.org/wiki/Random_forest">Random Forest algorithm</a> with a 22-feature vector of domain string characteristics.  Most of these features are numeric and are quick to calculate.  Our model is currently trained off-line on a corpus of highly malicious domains gathered from DNS traffic originating from a malware execution sandbox and benign, popular domains from a high traffic DNS sensor.  For stream classification, we use an internally developed platform for distributed high speed event processing that was built over Twitter's recently open sourced <a href="http://storm-project.net/">Storm project</a>. We discuss the system architecture as well as the logic behind our model's features and sampling techniques that have led to 97% classification accuracy on our dataset and the model's performance within our streaming environment.
</div></blockquote>
<br /><br />
Here are the <a href="http://www.slideshare.net/jasontrost/flo-con-clairvoyant-squirrel-final">slides</a> in case you're interested.
<br /><br />
--Jason