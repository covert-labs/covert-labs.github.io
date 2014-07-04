---
layout: post
title: Accumulo, Nutch, and Gora
description: "Accumulo, Nutch, and Gora"
modified: 2012-02-27
tags: [hadoop, accumulo, nutch, gora, web crawling]
image:
  feature: abstract-3.jpg
comments: true
share: true
---

In this post I will outline the steps necessary to use <a href="http://incubator.apache.org/accumulo/" target="_self">Accumulo</a> and <a href="http://gora.apache.org/" target="_self">Gora</a> to store content retrieved by <a href="http://nutch.apache.org/" target="_self">Nutch</a>.

###Apache Accumulo

<img alt="Accumulo Logo" src="/images/accumulo-logo.png" />

For those of you unfamiliar with Accumulo, it is an incubating Apache project and ...
> "Accumulo is a sorted, distributed key/value store based on Google's <a href="http://labs.google.com/papers/bigtable.html" title="BigTable">BigTable</a> design. It is built on top of Apache <a href="http://hadoop.apache.org/" title="Hadoop">Hadoop</a>, <a href="http://zookeeper.apache.org/" title="Zookeeper">Zookeeper</a>, and <a href="http://thrift.apache.org/" title="Thrift">Thrift</a>. It features a few novel improvements on the BigTable design in the form of cell-level access labels and a server-side programming mechanism that can modify key/value pairs at various points in the data management process."

Accumulo is conceptually very similar to HBase, but it has some nice features that HBase is currently lacking.  Some of these features are:

*   Cell level security
*   No fat row problem - i.e. entire rows don't need to fit in RAM
*   No limitation on Column Families or when Column Families can be created
*   Server side, data local, programming abstraction called Iterators.  Iterators are incredibly useful for adding functionality to Tablet Servers such as data local filtering, aggregation, and search.  
*   Check out this example application that shows off some of the features mentioned above: <a href="http://incubator.apache.org/accumulo/example/wikisearch.html" target="_self">THE WIKIPEDIA SEARCH EXAMPLE EXPLAINED, WITH PERFORMANCE NUMBERS.</a>

###Apache Gora
<img align="middle" alt="Gora Logo" height="158" src="/images/gora-logo.png" width="136" />

Gora is a object relational/non-relational mapping for arbitrary data stores including both relational (MySQL) and non-relational data stores (HBase, Cassandra, Accumulo, etc).  It was designed for Big Data applications and has support (interfaces) for Apache Pig, Apache Hive, Cascading, and generic Map/Reduce.

### Apache Nutch
<img align="middle" alt="Nutch Logo" height="113" src="/images/nutch-logo.png" width="300" />

Nutch is a highly scalable web crawler built over Hadoop Map/Reduce.  It was designed from the ground up to be an Internet scale web crawler.  This is a great overview of Nutch's architecture: <a href="http://www.slideshare.net/abial/nutch-as-a-web-data-mining-platform" target="_self">Nutch as a Web Data Mining Platform</a>

###Accumulo + Nutch + Gora

I generally prefer git over svn, in this post I use the source code hosted on github. 

####1. Obtain all sources (and Accumulo patch for GORA)   

{% highlight bash %}
git clone git://github.com/apache/nutch.git
git clone git://github.com/apache/gora.git
git clone git://github.com/apache/accumulo.git
wget https://issues.apache.org/jira/secure/attachment/12507986/GORA-65-1.patch
{% endhighlight %}

####2. Configure and Build Each Project:

#####Standard build and maven install for accumulo

{% highlight bash %}
cd accumulo
mvn package install
{% endhighlight %}

#####Patching GORA for Accumulo Support

Gora needs to be patched for support for Accumulo.  This patch should be considered beta, but I found it works good enough for experimenting with Nutch/GORA.  Note: ```-DskipTests``` is used because some of the tests seemed to hang indefinitely, so I skipped them for now.

{% highlight bash %}
cd ../gora
patch -p0 < ../GORA-65-1.patch
mvn package install -DskipTests
{% endhighlight %}

#####Building Nutch/GORA

So, getting Nutch/GORA to build was a bit of a challenge.  I will outline some of the hoops I had to jump through.  File paths mentioned below are assuming you are in the nutch project directory.

Run the following commands to checkout the nutchgora branch of Nutch.

{% highlight bash %}
cd ../nutch
git checkout origin/nutchgora
{% endhighlight %}

*   Modify the ```ivy/ivy.xml``` file. Change gora-core and gora-sql dependencies rev from ```"0.1.1-incubating"``` to ```"0.2-SNAPSHOT"```.  This is to match the patched version we just installed.  Also, add the following lines:
<script src="https://gist.github.com/1923654.js"></script>
*   Modify the ```ivy/ivysettings.xml``` file.  Add the following line to the top of the ```<resolvers>``` section. This will configure ant/ivy to use your local maven repository when resolving dependencies.  This is necessary because the patched version of GORA and the latest Accumulo version are not in any public maven repos.
<br />

{% highlight xml %}
<ibiblio name="local" root="file://${user.home}/.m2/repository/"
pattern="${maven2.pattern.ext}"  m2compatible="true"  />
{% endhighlight %}

<br />
<ul>
<li>Patch <code>src/java/org/apache/nutch/storage/StorageUtils.java</code></li>
<script src="https://gist.github.com/jt6211/1779397.js"></script>

<li>Create the file <code>conf/gora-accumulo-mapping.xml</code> with the following contents:</li>
<script src="https://gist.github.com/1923420.js"></script>

<li>Edit <code>conf/gora.properties</code> and add the following lines:</li>
<script src="https://gist.github.com/1923436.js"></script>

<li>edit (or create) <code>conf/nutch-site.xml</code> and adding the following property setting to it</li>
<script src="https://gist.github.com/1923513.js"></script>

<li>edit $HOME/.ivy2/cache/jaxen/jaxen/ivy-1.1.3.xml and find and comment out the following lines.  If anyone knows a more elegant way to accomplish this please let me know.  </li>
<script src="https://gist.github.com/1932096.js"></script>

If I don't comment out those dependencies, I get this error during compilation:
<script src="https://gist.github.com/1932108.js"></script>

<li>Run the following commands:</li>
</ul>

{% highlight bash %}
ant
{% endhighlight %}

<br />
####3. Deploy and Run

#####Configure Accumulo and its Dependencies

For this post I am only going to cover the basics for getting these systems to run on a single machine.  Deploying and running over a cluster may be covered in another post.

######Configure and Start Hadoop

{% highlight bash %}
cd ..
wget ftp://apache.cs.utah.edu/apache.org//hadoop/common/hadoop-0.20.2/hadoop-0.20.2.tar.gz
tar zxvf hadoop-0.20.2.tar.gz
{% endhighlight %}

Add the following to ```hadoop-0.20.2/conf/core-site.xml```

{% highlight xml %}
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://127.0.0.1/</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/home/${user.name}/tmp/hadoop</value>
  </property>
</configuration>
{% endhighlight %}

Add the following to ```hadoop-0.20.2/conf/mapred-site.xml```    

{% highlight xml %}
<configuration>
  <property>
    <name>mapred.job.tracker</name>
    <value>127.0.0.1:9001</value>
  </property>
</configuration>
{% endhighlight %}

Set the JAVA_HOME variable in ```hadoop-0.20.2/conf/hadoop-env.sh```.  Here is an example from my system:

{% highlight bash %}
export JAVA_HOME=/usr/lib/jvm/java-6-sun-1.6.0.26
{% endhighlight %}

At this point, you should have a bare bones configured hadoop installation.  It is time to start it...

Run the following commands:

{% highlight bash %}
mkdir -p ~/tmp/hadoop
cd hadoop-0.20.2
./bin/hadoop namenode -format
./bin/start-all.sh
{% endhighlight %}

If you configured everything properly, you should be able to open http://127.0.0.1:50070/dfshealth.jsp in a web browser and see a page that looks like this.  If there is a message saying that the Namenode is in safe mode, wait a minute or two and refresh the page.  It should go away.

![](/images/dfs-health.png)

You should also be able to open http://127.0.0.1:50030/jobtracker.jsp in a web browser and see a page that looks like this:

![](/images/mapred-health.png)

In both of these status webpages you should be able to see a ```1``` listed after ```"Live Nodes"``` and ```"Nodes"``` respectively.

######Configure and Start Zookeeper

{% highlight bash %}
cd ..
wget ftp://apache.cs.utah.edu/apache.org/zookeeper/zookeeper-3.4.3/zookeeper-3.4.3.tar.gz
tar zxvf zookeeper-3.4.3.tar.gz
{% endhighlight %}

Add the following to ```zookeeper-3.4.3/conf/zoo.cfg```.  Create this file if it does not exist.  **NOTE**: replace ```_USERNAME_``` with your username.

{% highlight bash %}
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
dataDir=/home/_USERNAME_/tmp/zookeeper-data
# the port at which the clients will connect
clientPort=2181
maxClientCnxns=100
{% endhighlight %}

Edit bin/zkEnv.sh.  Right after the following lines.

{% highlight bash %}
ZOOBINDIR=${ZOOBINDIR:-/usr/bin}
ZOOKEEPER_PREFIX=${ZOOBINDIR}/..
{% endhighlight %}

Add this line:

{% highlight bash %}
export ZOO_LOG_DIR=${ZOOKEEPER_PREFIX}/logs
{% endhighlight %}


{% highlight bash %}
cd zookeeper-3.4.3
export ZOO_LOG_DIR=$HOME/blogpost/zookeeper-3.4.3/logs
mkdir $ZOO_LOG_DIR
mkdir ~/tmp/zookeeper-data
{% endhighlight %}

At this point, you should have a configured zookeeper installation, it's time to start it...

{% highlight bash %}
./bin/zkServer.sh start
{% endhighlight %}

If you configured zookeeper and it successfully started you should be able to run the following command:

{% highlight bash %}
./bin/zkCli.sh -server 127.0.0.1:2181
{% endhighlight %}

It will output a bunch of logging messages, this is fine.  Press ```<ENTER>```, and then you should be dropped into a shell this a prompt that looks like the following: 

{% highlight text %}
[zk: 127.0.0.1:2181(CONNECTED) 0]
{% endhighlight %}

Type ```ls /``` and then ```<ENTER>```.  You should see a single line of output (followed again by a prompt) that looks like the following:

{% highlight bash %}
[zookeeper]
{% endhighlight %}

If so, then zookeeper is configured and running properly.  When you first run the zkCli.sh command, if you see stack traces that look like this:

{% highlight java %}
java.net.ConnectException: Connection refused
    at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
    at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:701)
    at org.apache.zookeeper.ClientCnxnSocketNIO.doTransport(ClientCnxnSocketNIO.java:286)
    at org.apache.zookeeper.ClientCnxn$SendThread.run(ClientCnxn.java:1035)
{% endhighlight %}

Then it means zookeeper failed to start for some reason, isn't listening on 127.0.0.1:2181, or you may have a local firewall blocking access to that port.

######Configure and Start Accumulo

{% highlight bash %}
cd ../accumulo
mvn package -P assemble
cp src/assemble/target/accumulo-1.5.0-incubating-SNAPSHOT-dist.tar.gz ../
cd ../
tar zxf accumulo-1.5.0-incubating-SNAPSHOT-dist.tar.gz
cd accumulo-1.5.0-incubating-SNAPSHOT/conf
rename s/.example// *.example
# basically disabling the custom security policy file for now
# since I had issues getting accumulo to work with it enabled
mv accumulo.policy accumulo.policy.example 
{% endhighlight %}

Edit ```accumulo-env.sh```.  At the top of the file, define HADOOP_HOME, ZOOKEEPER_HOME, and JAVA_HOME.  Here is an example:

{% highlight bash %}
export HADOOP_HOME=$HOME/hadoop-0.20.2
export ZOOKEEPER_HOME=$HOME/zookeeper-3.4.3
export JAVA_HOME=/usr/lib/jvm/java-6-sun-1.6.0.26
{% endhighlight %}

Edit ```accumulo-site.xml```.  

Set the ```logger.dir.walog``` property.

{% highlight xml %}
<property>
  <name>logger.dir.walog</name>
  <value>$HOME/tmp/walogs</value>
</property>
{% endhighlight %}

Set the ```instance.secret``` property.

{% highlight xml %}
<property>
  <name>instance.secret</name>
  <value>SOME-PASSWORD-OF-YOUR-CHOOSING</value>
</property>
{% endhighlight %}

Run the following commands:

{% highlight bash %}
mkdir $HOME/tmp/walogs
cd ../
{% endhighlight %}

At this point you should have a fully configured Accumulo installation.  It is time to initialize it and start it...

{% highlight bash %}
./bin/accumulo init
{% endhighlight %}

You should see similar output to this.  I set my instance name to ```"inst"``` and my password to ```"secret"```.  You may want to do the same for the sake of this tutorial or make sure to set the correct config parameters later.

{% highlight text %}
23 08:15:26,635 [util.Initialize] INFO : Hadoop Filesystem is hdfs://127.0.0.1/
23 08:15:26,637 [util.Initialize] INFO : Accumulo data dir is /accumulo
23 08:15:26,637 [util.Initialize] INFO : Zookeeper server is localhost:2181

Warning!!! Your instance secret is still set to the default, this is not secure. 
We highly recommend you change it.

You can change the instance secret in accumulo by using: bin/accumulo 
org.apache.accumulo.server.util.ChangeSecret oldPassword newPassword. 
You will also need to edit your secret in your configuration file by adding the property 
instance.secret to your conf/accumulo-site.xml. Without this accumulo will not operate correctly
Instance name : inst
Enter initial password for root: ******
Confirm initial password for root: *****
23 08:15:34,100 [util.NativeCodeLoader] INFO : Loaded the native-hadoop library
23 08:15:34,337 [security.ZKAuthenticator] INFO : Initialized root user with 
username: root at the request of user !SYSTEM
{% endhighlight %}

**Note**: If it appears to hang after you entered the instance name, zookeeper may not be running.  ```<CTRL>-C``` the accumulo init and make sure zookeeper is running.

Now run:

{% highlight bash %}
./bin/start-all.sh
{% endhighlight %}

After this finishes, you should be able to open http://127.0.0.1:50095/ in a web browser and see a page very similar to this:

![](/images/accumulo-webapp.png)

The important items to note on this page are the is a 1 after the Tablet Servers, Live Data Nodes, and Trackers in the "Accumulo Master", "NameNode", and "JobTracker" tables, respectively.  There should also be entry in the "Zookeeper" table.

###4. Crawl

At this point, you should have a fully functional Hadoop, Zookeeper, and Accumulo install, so we are ready to run a Nutch web crawl.  Create a file with URLs, one per line, call it seeds.txt and place it in your home directory.  I added the following URLs to my seeds file:

{% highlight text %}
http://projects.apache.org/indexes/alpha.html
http://www.dmoz.org/Arts/People/
{% endhighlight %}

Run the following commands:
     
{% highlight bash %}
cd ../nutch/
./runtime/local/bin/nutch crawl file://$HOME/seeds.txt -depth 1
{% endhighlight %}

You should see some log messages printed to the console, but hopefully no stack traces.  If you see a stack trace, you may need to go back and check your configs to make sure they match the ones we created earlier.

After the crawler finishes, you should be able to explore it using the accumulo shell.
     
{% highlight bash %}
cd ../accumulo-1.5.0-incubating-SNAPSHOT/bin/
./accumulo shell -u root -p secret
{% endhighlight %}

<script src="https://gist.github.com/1928543.js"></script>

Further details and exploration of this data in Accumulo will have to wait for another blog post.

I ended up posting all the modified code from Gora (accumulo patch) and Nutchgora (patches for getting gora-accumulo working) to my github.  Check it out.

* [Nutchgora + Accumulo branch](https://github.com/jt6211/nutch/tree/gora-accumulo)
* [Gora + Accumulo branch](https://github.com/jt6211/gora/tree/gora-accumulo)<br />

Let me know if you have any questions...

--Jason<br />
[@jason_trost](https://twitter.com/#!/jason_trost)

**Update (3/2):** someone told me that they had problems getting nutch to build and that this [patch](https://issues.apache.org/jira/secure/attachment/12460512/GORA-18-patch.txt) worked for them (even though the patch is for GORA).  I would be curious if anyone else has this same issue.  Here is the error they encountered when building with ```ant```:


{% highlight text %}        
[ivy:resolve] :: problems summary ::
[ivy:resolve] :::: WARNINGS
[ivy:resolve]           ::::::::::::::::::::::::::::::::::::::::::::::
[ivy:resolve]           ::          UNRESOLVED DEPENDENCIES         ::
[ivy:resolve]           ::::::::::::::::::::::::::::::::::::::::::::::
[ivy:resolve]           :: log4j#log4j;1.2.15: configuration not found in log4j#log4j;1.2.15: 'master'.
{% endhighlight %}
 