---
layout: post
title: "Hetrogeneus Information Networks + Cyber Security Use cases"
description: ""
modified: 2019-07-18

image:
  feature: grey-scale-dark.jpg
comments: true
share: true
---

This post explores Hetrogeneus Information Networks (HIN) and applications to Cyber security.  

Over the past few months I have been researching Hetrogeneus Information Networks (HIN) and Cyber security use cases.  I first encountered HIN's after discovering this paper: ["Gotcha: Sly Malware!- Scorpion A Metagraph2vec Based Malware Detection System"](/research-papers/hetrogeneus-information-networks/Gotcha - Sly Malware!- Scorpion A Metagraph2vec Based Malware Detection System.pdf) through a Google Scholar Alert I had setup for ["Guilt by Association: Large Scale Malware Detection by Mining File-relation Graphs"](/research-papers/hetrogeneus-information-networks/Guilt by Association - Large Scale Malware Detection by Mining File-relation Graphs.pdf).  If you're interested in how I setup my Google Alerts to stay abreast of the latest security data science research, see this: [Security Data Science Learning Resources](https://medium.com/@jason_trost/security-data-science-learning-resources-8f7586995040).

Hetrogeneus Information Networks are a relatively simple way of modelling a dataset as a graph made of nodes and edges where 1) all nodes and edges have defined types, and 2) types of nodes > 1 or types of edges > 1 (hence "Heterogeneus").  The set of node and edge types represents the schema of the network.  This differs from homogeneuos networks where the nodes and edges are all the same type (e.g. Facebook Social Network, Website hyperlinks).  HINs provide a very rich abstraction for modelling complex datasets.

Below is an example schema taken from the [HinDom paper](/research-papers/hetrogeneus-information-networks/HinDom- A Robust Malicious Domain Detection System based on Heterogeneous Information Network with Transductive Classification.pdf).

![HinDom Schema](/images/hin/hindom-schema-2.png)

This schema represents three combined datasets (Passive DNS, DNS query logs, DNS response logs) and it models three node types (Client, Domain, and IP Address) and six edge types (segment, query, CNAME, similar, resolve, and same-domain).  Here is an expanded example and descriptions of the relationships:

![HinDom Example](/images/hin/hindom-example.png)

* **Client-query-Domain** - matrix Q denotes that domain i is queried by client j.
* **Client-segment-Client** - matrix N denotes that client i and client j belong to the same network segment. 
* **Domain-resolve-IP** - matrix R denotes that domain i is resolved to IP address j.
* **Domain-similar-Domain** - matrix S denotes the character-level similarity between domain i and j.
* **Domain-cname-Domain** - matrix C denotes that domain i and domain j are in a CNAME record.
* **IP-domain-IP** - matrix D denotes that IP address i and IP address j are once mapped to the same domain.


Once the dataset is represented as a graph, it needs to be converted into feature vectors before machine learning models can be built.  A common technique for featurizing a HIN is by defining Meta-paths or Meta-graphs against the graph and then performing guided random walks against the defined meta-paths/graphs.  Meta-paths represent graph traversals through specific node and edge sequences.  Meta-paths are akin to feature engineering in classical machine learning as it is very important to select meta-paths that provide useful signals for whatever variable is being predicted.  These random walks against meta-paths produce a sequence of nodes, similar to sentences of tokens, which can then be fed into models like [Skipgram](https://arxiv.org/pdf/1301.3781.pdf) to create embeddings.  Once the nodes are represented as embeddings many different models (SVM, DNN, etc) can be used to solve many different types of problems (Simiarity Search, Classification, Clustering, Recommendation, etc).  Below are the metapaths used in the HinDom paper.

![HinDom Metapaths](/images/hin/hindom-metapaths.png)

Below are some resources that I found useful for learning more about Heterogeneous Information Networks.

### Books:

* [Mining Heterogeneous Information Networks: Principles and Methodologies](https://www.amazon.com/Mining-Heterogeneous-Information-Networks-Methodologies/dp/1608458806/ref=as_li_ss_tl?ie=UTF8&linkCode=ll1&tag=cyberanaly-20&linkId=7f761b6fc9b4e6a799a8d71d9e018cbb&language=en_US)
* [Heterogeneous Information Network Analysis and Applications](https://www.amazon.com/Heterogeneous-Information-Analysis-Applications-Analytics-ebook/dp/B071P9W8JV/ref=as_li_ss_tl?ie=UTF8&linkCode=ll1&tag=cyberanaly-20&linkId=3b1edf38484828593ed6a0faad474d4a&language=en_US) 

### HIN Papers: 

* [Mining Heterogeneous Information Networks- A Structural Analysis Approach](/research-papers/hetrogeneus-information-networks/Mining Heterogeneous Information Networks- A Structural Analysis Approach.pdf)
* [HIN2Vec: Explore Meta-paths in Heterogeneous Information Networks for Representation Learning](/research-papers/hetrogeneus-information-networks/HIN2Vec- Explore Meta-paths in Heterogeneous Information Networks for Representation Learning.pdf)
* [PathSim: Meta Path-Based Top-K Similarity Search in Heterogeneous Information Networks](/research-papers/hetrogeneus-information-networks/PathSim- Meta Path-Based Top-K Similarity Search in Heterogeneous Information Networks.pdf)
* [Ranking-Based Clustering of Heterogeneous Information Networks with Star Network Schema](/research-papers/hetrogeneus-information-networks/Ranking-Based Clustering of Heterogeneous Information Networks with Star Network Schema.pdf)
* [Metapath2vec: Scalable Representation Learning for Heterogeneous Networks](/research-papers/hetrogeneus-information-networks/metapath2vec- Scalable Representation Learning for Heterogeneous Networks.pdf)
* [A Survey of Heterogeneous Information Network Analysis](/research-papers/hetrogeneus-information-networks/A Survey of Heterogeneous Information Network Analysis.pdf)
* [Adversarial Learning on Heterogeneous Information Networks](/research-papers/hetrogeneus-information-networks/Adversarial Learning on Heterogeneous Information Networks.pdf)

### Security-related HIN Papers:

#### Malware Detection / Code Analysis:

* [AiDroid: When Heterogeneous Information Network Marries Deep Neural Network for Real-time Android Malware Detection](/research-papers/hetrogeneus-information-networks/AiDroid - When Heterogeneous Information Network Marries Deep Neural Network for Real-time Android Malware Detection.pdf)
* [Gotcha: Sly Malware!- Scorpion A Metagraph2vec Based Malware Detection System](/research-papers/hetrogeneus-information-networks/Gotcha - Sly Malware!- Scorpion A Metagraph2vec Based Malware Detection System.pdf)
* [HinDroid: An Intelligent Android Malware Detection System Based on Structured Heterogeneous Information Network](/research-papers/hetrogeneus-information-networks/HinDroid - An Intelligent Android Malware Detection System Based on Structured Heterogeneous Information Network.pdf)
* [Make Evasion Harder: An Intelligent Android Malware Detection System](/research-papers/hetrogeneus-information-networks/Make Evasion Harder- An Intelligent Android Malware Detection System.pdf)
* [DeepAM: a heterogeneous deep learning framework for intelligent malware detection](https://link.springer.com/article/10.1007/s10115-017-1058-9)
* [HinDom: A Robust Malicious Domain Detection System based on Heterogeneous Information Network with Transductive Classification](/research-papers/hetrogeneus-information-networks/HinDom- A Robust Malicious Domain Detection System based on Heterogeneous Information Network with Transductive Classification.pdf)
* [iTrustSO: An Intelligent System for Automatic Detection of Insecure Code Snippets in Stack Overflow](/research-papers/hetrogeneus-information-networks/iTrustSO - An Intelligent System for Automatic Detection of Insecure Code Snippets in Stack Overflow.pdf)


#### Mining the Darkweb / Fraud Detection / Social Network Analysis:

* [Key Player Identification in Underground Forums over Attributed Heterogeneous Information Network Embedding Framework](/research-papers/hetrogeneus-information-networks/Key Player Identification in Underground Forums over Attributed Heterogeneous Information Network Embedding Framework.pdf)
* [Your Style Your Identity: Leveraging Writing and Photography Styles for Drug Trafficker Identification in Darknet Markets over Attributed Heterogeneous Information Network](/research-papers/hetrogeneus-information-networks/Your Style Your Identity- Leveraging Writing and Photography Styles for Drug Trafficker Identification in Darknet Markets over Attributed Heterogeneous Information Network.pdf)
* [iDetector: Automate Underground Forum Analysis Based on Heterogeneous Information Network](/research-papers/hetrogeneus-information-networks/iDetector - Automate Underground Forum Analysis Based on Heterogeneous Information Network.pdf)
* [Cash-out User Detection based on Attributed Heterogeneous Information Network with a Hierarchical Attention Mechanism](/research-papers/hetrogeneus-information-networks/Cash-out User Detection based on Attributed Heterogeneous Information Network with a Hierarchical Attention Mechanism.pdf)
* [iDev: Enhancing Social Coding Security by Cross-platform User Identification Between GitHub and Stack Overflow](/research-papers/hetrogeneus-information-networks/iDev- Enhancing Social Coding Security by Cross-platform User Identification Between GitHub and Stack Overflow.pdf)

### Tutorials:

* [KDD 2017: Mining Heterogeous Information Networks](http://web.cs.ucla.edu/~yzsun/Tutorials/KDD2017/KDD_17_Recommendation.pdf)
* [Intro to Heterogeneous (Information) Networks](http://people.cs.vt.edu/~badityap/classes/cs6604-Fall17/student-lectures/prashant-hetero-networks.pdf)

### Code:

* https://github.com/zhoushengisnoob/HINE - Heterogeneous Information Network Embedding: papers and code implementations.
* https://github.com/hetio/hetnetpy - HIN library
* https://github.com/hetio/hetmatpy - HIN library that represents as matrices.
* https://github.com/stellargraph/stellargraph (see [stellargraph-metapath2vec.ipynb](https://github.com/stellargraph/stellargraph/blob/develop/demos/embeddings/stellargraph-metapath2vec.ipynb))
* https://github.com/csiesheep/hin2vec

### Prominent Security Researchers using HIN:

* [Yanfang Ye](https://scholar.google.com/citations?user=egjr888AAAAJ&hl=en&oi=ao)
* [Shifu Hou](https://scholar.google.com/citations?hl=en&user=-NnGknEAAAAJ)
* [Yiming Zhang](https://scholar.google.com/citations?hl=en&user=0fTVEgQAAAAJ)


---

As always, feedback is welcome so please leave a message here, on [Medium](https://medium.com/@jason_trost), or @ me on [twitter]((https://twitter.com/#!/jason_trost))!

--Jason
<br />[@jason_trost](https://twitter.com/#!/jason_trost)

