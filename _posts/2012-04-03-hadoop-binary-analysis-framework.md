---
layout: post
title: Hadoop Binary Analysis Framework
description: "Large-scale Malware Static Analysis using Hadoop"
modified: 2013-08-01
tags: [dns, data mining, hadoop, mapreduce, malware, static analysis]
image:
  feature: abstract-7.jpg
comments: true
share: true
permalink: /post/20404247709/hadoop-binary-analysis-framework
---

**Update (2013-08-01)**:  This project is no longer maintain since we port all this functionality over to BinaryPig. Use [BinaryPig](https://github.com/endgameinc/binarypig) instead.  For more information on BinaryPig, see [Slides](http://www.slideshare.net/jasontrost/binary-24851796), [Paper](https://media.blackhat.com/us-13/US-13-Hanif-Binarypig-Scalable-Malware-Analytics-in-Hadoop-WP.pdf), or [Video](https://www.youtube.com/watch?v=fcLkTvnBpIw).

---

This is a quick post.  I wrote this little framework for using Hadoop to analyze lots of small files.  This may not be the most optimal way of doing this, but it worked well and makes repeated analysis tasks easy and scalable.

[https://github.com/jt6211/hadoop-binary-analysis](https://github.com/jt6211/hadoop-binary-analysis)

I recently needed a quick way to analyze millions of small binary files (from 100K-19MB each) and
I wanted a scalable way to repeatedly do this sort of analysis.  I chose Hadoop as the platform,
and I built this little framework (really, a single MapReduce job) to do it.  This is very much a 
work in progress, and feedback and pull requests are welcome.

The main MapReduce job in this framework accepts a Sequence file of ```<Text, BytesWritable>``` where the 
```Text``` is a name and the ```BytesWritable``` is the contents of a file.  The framework unpacks the bytes of 
the ```BytesWritable``` to the local filesystem of the mapper it is running on, allowing the mapper to run
arbitrary analysis tools that require local filesystem access.  The framework then captures stdout and stderr from the
analysis tool/script and stores it (how it stores it is pluggable, see ```io.covert.binary.analysis.OutputParser```).

Building:

{% highlight bash %}
mvn package assembly:assembly
{% endhighlight %}

Running:
    
{% highlight bash %}
JAR=target/hadoop-binary-analysis-1.0-SNAPSHOT-job.jar

# a local directory with files in it (directories are ignored for now)
LOCAL_FILES=src/main/java/io/covert/binary/analysis/
INPUT="dir-in-hdfs"
OUTPUT="output-dir-in-hdfs"

# convert a bunch of relatively small files into one sequence file (Text, BytesWritable)
hadoop jar $JAR io.covert.binary.analysis.BuildSequenceFile $LOCAL_FILES $INPUT

# Use the config properties in example.xml to basically run the wrapper.sh script on each file using Hadoop
# as the platform for computation
hadoop jar $JAR io.covert.binary.analysis.BinaryAnalysisJob -files wrapper.sh -conf example.xml $INPUT $OUTPUT
{% endhighlight %}

From example.xml:

{% highlight xml %}
<property>
  <name>binary.analysis.program</name>
  <value>./wrapper.sh</value>
</property>
<property>
  <name>binary.analysis.program.args</name>
  <value>${file}</value>
</property>
<property>
  <name>binary.analysis.program.args.delim</name>
  <value>,</value>
</property>
{% endhighlight %}

<br />
This block of example instructs the framework to run ```wrapper.sh``` using the args of ```${file}``` (where ```${file}```
is replaced by the unpacked filename from the Sequence File.  If multiple command line args are required,
they can be specified by appending a delimiter and then each arg to the value of the ```binary.analysis.program.args```
property


--Jason
