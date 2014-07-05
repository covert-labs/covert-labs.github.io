---
layout: post
title: Accumulo and Pig
description: "Accumulo and Pig"
modified: 2012-03-02
tags: [hadoop, apache pig, accumulo, bigdata]
image:
  feature: abstract-3.jpg
comments: true
share: true
permalink: /post/18605091231/accumulo-and-pig
---

I just posted a functional AccumuloStorage module to [github](https://github.com/jt6211/accumulo-pig).

Here's how you use it (also in the github README)

###1. Build the JAR 

**Note:** you will need to download the accumulo src, build it, and install it into your maven repo before this will work

{% highlight bash %}
mvn package
{% endhighlight %}

This will create a JAR file here: ```target/accumulo-pig-1.5.0-incubating-SNAPSHOT.jar```

###2. Download the JARs needed by pig

{% highlight bash %}
mvn dependency:copy-dependencies -DoutputDirectory=lib  \
    -DincludeArtifactIds=zookeeper,libthrift,accumulo-core,cloudtrace
{% endhighlight %}    

This should have copied the needed dependency jars into a ```lib``` directory.

###3. Print the register statements we will need in pig

{% highlight bash %}
for JAR in lib/*.jar target/accumulo-pig-1.5.0-incubating-SNAPSHOT.jar ; 
do 
    echo register `pwd`/$JAR; 
done
{% endhighlight %}

Here is some example output

{% highlight bash %}
register /home/developer/workspace/accumulo-pig/lib/accumulo-core-1.5.0-incubating-SNAPSHOT.jar
register /home/developer/workspace/accumulo-pig/lib/cloudtrace-1.5.0-incubating-SNAPSHOT.jar
register /home/developer/workspace/accumulo-pig/lib/libthrift-0.6.1.jar
register /home/developer/workspace/accumulo-pig/lib/zookeeper-3.3.1.jar
register /home/developer/workspace/accumulo-pig/target/accumulo-pig-1.5.0-incubating-SNAPSHOT.jar
{% endhighlight %}

####5. Run Pig

Copy the register statements above and paste them into the pig terminal.  Then you can LOAD from and STORE into accumulo.

{% highlight bash %}
$ pig
2012-03-02 08:15:25,808 [main] INFO  org.apache.pig.Main - Logging error messages to: /home/developer/workspace/accumulo-pig/pig_1330694125807.log
2012-03-02 08:15:25,937 [main] INFO  org.apache.pig.backend.hadoop.executionengine.HExecutionEngine - Connecting to hadoop file system at: hdfs://127.0.0.1/
2012-03-02 08:15:26,032 [main] INFO  org.apache.pig.backend.hadoop.executionengine.HExecutionEngine - Connecting to map-reduce job tracker at: 127.0.0.1:9001
grunt> register /home/developer/workspace/accumulo-pig/lib/accumulo-core-1.5.0-incubating-SNAPSHOT.jar
grunt> register /home/developer/workspace/accumulo-pig/lib/cloudtrace-1.5.0-incubating-SNAPSHOT.jar
grunt> register /home/developer/workspace/accumulo-pig/lib/libthrift-0.6.1.jar
grunt> register /home/developer/workspace/accumulo-pig/lib/zookeeper-3.3.1.jar
grunt> register /home/developer/workspace/accumulo-pig/target/accumulo-pig-1.5.0-incubating-SNAPSHOT.jar
grunt> 
grunt> DATA = LOAD 'accumulo://webpage?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181&columns=f:cnt' 
>>using org.apache.accumulo.pig.AccumuloStorage() AS (row, cf, cq, cv, ts, val);
grunt> 
grunt> DATA2 = FOREACH DATA GENERATE row, cf, cq, cv, val;
grunt> 
grunt> STORE DATA2 into 'accumulo://webpage_content?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181' using org.apache.accumulo.pig.AccumuloStorage();
2012-03-02 08:18:44,090 [main] INFO  org.apache.pig.tools.pigstats.ScriptState - Pig features used in the script: UNKNOWN
2012-03-02 08:18:44,093 [main] INFO  org.apache.pig.newplan.logical.rules.ColumnPruneVisitor - Columns pruned for DATA: $4
2012-03-02 08:18:44,108 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MRCompiler - File concatenation threshold: 100 optimistic? false
2012-03-02 08:18:44,110 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MultiQueryOptimizer - MR plan size before optimization: 1
2012-03-02 08:18:44,110 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MultiQueryOptimizer - MR plan size after optimization: 1
2012-03-02 08:18:44,117 [main] INFO  org.apache.pig.tools.pigstats.ScriptState - Pig script settings are added to the job
2012-03-02 08:18:44,118 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.JobControlCompiler - mapred.job.reduce.markreset.buffer.percent is not set, set to default 0.3
2012-03-02 08:18:44,120 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.JobControlCompiler - creating jar file Job7611629033341757288.jar
2012-03-02 08:18:46,282 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.JobControlCompiler - jar file Job7611629033341757288.jar created
2012-03-02 08:18:46,286 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.JobControlCompiler - Setting up single store job
2012-03-02 08:18:46,375 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - 1 map-reduce job(s) waiting for submission.
2012-03-02 08:18:46,876 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - 0% complete
2012-03-02 08:18:46,878 [Thread-17] INFO  org.apache.pig.backend.hadoop.executionengine.util.MapRedUtil - Total input paths (combined) to process : 1
2012-03-02 08:18:47,887 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - HadoopJobId: job_201203020643_0001
2012-03-02 08:18:47,887 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - More information at: http://127.0.0.1:50030/jobdetails.jsp?jobid=job_201203020643_0001
2012-03-02 08:18:54,434 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - 50% complete
2012-03-02 08:18:57,484 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - 100% complete
2012-03-02 08:18:57,485 [main] INFO  org.apache.pig.tools.pigstats.SimplePigStats - Script Statistics: 
 
HadoopVersionPigVersionUserIdStartedAtFinishedAtFeatures
0.20.20.9.2developer2012-03-02 08:18:442012-03-02 08:18:57UNKNOWN
 
Success!
 
Job Stats (time in seconds):
JobIdMapsReducesMaxMapTimeMinMapTImeAvgMapTimeMaxReduceTimeMinReduceTimeAvgReduceTimeAliasFeatureOutputs
job_201203020643_000110333000DATA,DATA2MAP_ONLYaccumulo://webpage_content?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181,
 
Input(s):
Successfully read 288 records from: "accumulo://webpage?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181&columns=f:cnt"
 
Output(s):
Successfully stored 288 records in: "accumulo://webpage_content?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181"
 
Counters:
Total records written : 288
Total bytes written : 0
Spillable Memory Manager spill count : 0
Total bags proactively spilled: 0
Total records proactively spilled: 0
 
Job DAG:
job_201203020643_0001
 
 
2012-03-02 08:18:57,492 [main] INFO  org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.MapReduceLauncher - Success!
grunt> 
{% endhighlight %}

Here are the pig commands run if you don't want to look through the output above:

{% highlight java %}
# load just the web content (from the f:cnt column) from the webpage table
DATA = LOAD 
'accumulo://webpage?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181&columns=f:cnt' 
   using org.apache.accumulo.pig.AccumuloStorage() AS (row, cf, cq, cv, ts, val);

# basically, remove the ts field since it is not needed
DATA2 = FOREACH DATA GENERATE row, cf, cq, cv, val;

# store the data as is in a new table called webpage_content
STORE DATA2 into 
'accumulo://webpage_content?instance=inst&user=root&password=secret&zookeepers=127.0.0.1:2181' 
   using org.apache.accumulo.pig.AccumuloStorage();
{% endhighlight %}

A more detailed blog post going in more detail of how/why this is useful will follow.

--Jason

Update (2012/03/04): you may want to run this as the first line of the pig script:

{% highlight bash %}
SET mapred.map.tasks.speculative.execution false
{% endhighlight %}

This will avoid ingesting duplicate entries into accumulo.  For the data from this post, ingesting duplicate entries wouldn't cause any real issues because Accumulo's ```VersioningIterator``` would only keep the newest copy, but for columns/tables with aggregation configured (e.g. using ```LongCombiner```) we definitely don't want this.