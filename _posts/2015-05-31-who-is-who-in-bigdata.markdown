---
layout: post
title:  "Who is who in Big Data"
date:   2015-05-31 19:17:10
categories: BigData
comments: true
---

Almost year ago I've returned back to Big Data area and started overview of new technologies. I was impressed with a lot of new tools and libraries which I never heard before (I've used Hadoop few years ago). So let me share results of this overview, quick look on who is who in Big Data.

* TOC
{:toc}

### What is Big Data ###

*Big data is like teenage sex: everyone talks about it, nobody really knows how to do it, everyone thinks everyone else is doing it, so everyone claims they are doing it...*

For many people Big Data it's a magic buzzword which sounds attractive, but have no exact meaning, so we have to start with definition. 

IMHO this picture will be the good starting point to understand what is Big Data and Fast Data

![The 3Vs that define Big Data](/assets/posts/2015-05-31-who-is-who-in-bigdata/what-is-bigdata.jpg)

This image is from Diya Soubra's blog post, [The 3Vs that define Big Data](http://www.datasciencecentral.com/forum/topics/the-3vs-that-define-big-data) and I highly recommend to read it before you'll continue reading.

### Hadoop stack evolution ###

*The raging ocean that covered everything was engulfed in total darkness, and the Spirit of God was moving over the water. (Bible. Genesis)*

Most known Big Data technology is Hadoop, which initially was based on [Map-Reduce](http://static.googleusercontent.com/media/research.google.com/en/us/archive/mapreduce-osdi04.pdf) ([see also](http://en.wikipedia.org/wiki/MapReduce)) pattern [implementation engine](http://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html) and HDFS.  

Then, a lot of additional technologies, which can use or can be used with Hadoop, was created or adopted (like Pig, Hive etc.).  

So we can say that Hadoop now is the core of Big Data stack (I mean important and frequently used). But keep in mind that Hadoop **wasn't first** (see [BOINC](http://en.wikipedia.org/wiki/Berkeley_Open_Infrastructure_for_Network_Computing) for example) and Hadoop **isn't mandatory** part of Big Data stacks. Many frameworks and libraries can be used with or without Hadoop or even standalone.

Hadoop 2.0 introduced [YARN](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html), resource management and a central platform across Hadoop cluster.

![Hadoop Stack Evolution](/assets/posts/2015-05-31-who-is-who-in-bigdata/hadoopstack-evolution.png)

Image from Hortonworks [Apache Hadoop 2 is now GA!](http://hortonworks.com/blog/apache-hadoop-2-is-ga/) article, highly recommended to read it before you'll continue.

**NOTE:** When we are talking about Hadoop, keep in mind that it's open-source [Apache project](https://hadoop.apache.org/) with many contributors. Also we have many Hadoop Distributions (Hadoop + additional frameworks and libraries) created and supported by many companies. Most known are [Hortonworks](http://hortonworks.com) and [Cloudera](http://cloudera.com).

**NOTE:** In this article I'll use a lot of materials from [Hortonworks](http://hortonworks.com), but the only reason for this is good documentation. So, there are many other distributions with different content, this particular one was used just to illustrate basic principles. 

Let's look what is included to Hortonworks Distribution, for example

![Hortonworks Distribution](/assets/posts/2015-05-31-who-is-who-in-bigdata/hortonworks-distribution.png)

As you can see, it's not only Hadoop inside, but a lot of frameworks and libraries, which can be used on top or with Hadoop. 
But what is Hive, Pig etc.? Before we will dive in details let's look on the whole picture.  

I'll use one more image from [Hortonworks Data Platform description page](http://hortonworks.com/hdp/), where you can find more details.

![HDP](/assets/posts/2015-05-31-who-is-who-in-bigdata/hdp.jpg)


And now let's overview most important technologies

### List of Big Data technologies ###

This is my incomplete list of the technologies you should be at least aware about. You may not be familiar with them, but you definetly need to be aware about their existence and get basic understanding.


**NOTE:** Before you'll say that this list is unstructured, I'd like to note in the next section **you'll find schema which may be also good starting point**. So,

![KEEP CALM AND CONTINUE READING](/assets/posts/2015-05-31-who-is-who-in-bigdata/calm-and-read.jpg)


**Accumulo** (Apache) [BIGTABLE][NOSQL][KEYVALUE][STORAGE] - sorted, distributed key/value store based on the BigTable technology from Google. It is a system built on top of Apache Hadoop, Apache ZooKeeper, and Apache Thrift. Written in Java, Accumulo has cell-level access labels and server-side programming mechanisms. Accumulo is the 3rd most popular NoSQL Wide Column system according to DB-Engines ranking of Wide Column Stores  

**Drill** (Apache) (MapR) [INTERACTIVE] [SQL] - low latency SQL query engine for Hadoop and NoSQL, distributed system for interactive analysis of large-scale datasets.  

**Flume** (Apache) [EVENT INPUT] - distributed system for collecting log data from many sources, aggregating it, and writing it to HDFS  

**GraphX** (Apache) [ETL][REPRESENTATION] - is Apache Spark's API for graphs and graph-parallel computation.  

**H2O** - in-memory scalable machine learning framework ([http://h2o.ai](http://h2o.ai)). Can be used as standalone cluster or with Spark (see [Sparking Water project](http://0xdata.com/product/sparkling-water/))

**Hadoop** [MR ENGINE][PROCESSING ENGINE] - framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.  

**HBase** (Apache)  [BIGTABLE][NOSQL][KEYVALUE][STORAGE] -  Hadoop database, a distributed, scalable, big data store.  Open source, non-relational, distributed database modeled after Google's BigTable and written in Java. It is developed as part of Apache Software Foundation's Apache Hadoop project and runs on top of HDFS (Hadoop Distributed Filesystem), providing BigTable-like capabilities for Hadoop. That is, it provides a fault-tolerant way of storing large quantities of sparse data (small amounts of information caught within a large collection of empty or unimportant data, such as finding the 50 largest items in a group of 2 billion records, or finding the non-zero items representing less than 0.1% of a huge collection). HBase features compression, in-memory operation, and Bloom filters on a per-column basis as outlined in the original BigTable paper. Tables in HBase can serve as the input and output for MapReduce jobs run in Hadoop, and may be accessed through the Java API but also through REST, Avro or Thrift gateway APIs. It is columnar and provides fault-tolerant storage and quick access to large quantities of sparse data. It also adds transactional capabilities to Hadoop, allowing users to conduct updates, inserts and deletes.  

**HDFS** [DFS] - Java-based file system that provides scalable and reliable data storage that is designed to span large clusters of commodity servers.  

**Hive** [SQL][MR] - open-source data warehouse system for querying and analyzing large datasets stored in Hadoop files. Hive is not designed for OLTP workloads and does not offer real-time queries or row-level updates. It is best used for batch jobs over large sets of append-only data (like web logs). Hive is based on Hadoop and MapReduce operations, there are several key differences. The first is that Hadoop is intended for long sequential scans, and because Hive is based on Hadoop, you can expect queries to have a very high latency (many minutes).  

**Hue** (Cloudera) [WEBPANEL] - is an open source UI for Hadoop and its satellite  projects. It makes the whole platform (e.g. HDFS, MapReduce, Hive,  Oozie, Pig, Impala, Solr...) easy to use and accessible from your  browser (e.g. upload files to HDFS, send Hive queries from a Web editor,  build workflows with Drag & Drop... all within a single app)  

**Impala** (Cloudera) [INTERACTIVE SQL][SQL] - Impala project brings scalable parallel database technology to Hadoop, enabling users to issue low-latency SQL queries to data stored in HDFS and Apache HBase without requiring data movement or transformation. Impala is integrated with Hadoop to use the same file and data formats, metadata, security and resource management frameworks used by MapReduce, Apache Hive, Apache Pig and other Hadoop software.  

**Kafka** (Apache) [EVENT INPUT] - distributed publish-subscribe messaging system. It is designed to provide high throughput persistent messaging that’s scalable and allows for parallel data loads into Hadoop. Its features include the use of compression to optimize IO performance and mirroring to improve availability, scalability and to optimize performance in multiple-cluster scenarios.  

**Knox** (Apache) [SECURITY] [HADOOP] - Knox Gateways provides security for multiple Hadoop clusters, system that provides a single point of authentication and access for Apache™ Hadoop® services in a cluster  

**Mahout** (Apache) - [AI] library of scalable machine-learning algorithms, implemented on top of Apache Hadoop® and using the MapReduce paradigm. Included as sub-project to Lucene.  

**Lucene** (Apache) [SEARCH ENGINE] - The Apache Lucene project develops open-source software, including Lucene Core which provides Java-based indexing and search technology, Solr which is the high performance search server built for Lucene Core, Open Source Relevance Project and PyLucene, a Python port of the Core project.  

**MapReduce** (Google) - programming model and an associated implementation for processing and generating large data sets with a parallel, distributed algorithm on a cluster. [Google Article](http://static.googleusercontent.com/media/research.google.com/en/us/archive/mapreduce-osdi04.pdf), [Wikipedia](http://en.wikipedia.org/wiki/MapReduce)

**MLib** (Apache, Spark) [AI] - MLlib is a Spark subproject providing machine learning primitives. Initial contribution from AMPLab, UC Berkeley, shipped with Spark since version 0.8. MLib is a Spark implementation of some common machine learning algorithms and utilities, including classification, regression, clustering, collaborative filtering, dimensionality reduction, as well as underlying optimization primitives.  

**Oozie** (Apache) [SCHEDULER] - Java Web application used to schedule Apache Hadoop jobs. Oozie combines multiple jobs sequentially into one logical unit of work. It is integrated with the Hadoop stack and supports Hadoop jobs for Apache MapReduce, Apache Pig, Apache Hive, and Apache Sqoop. It can also be used to schedule jobs specific to a system, like Java programs or shell scripts.  

**Pig** (Apache) [TASK DEFINITION] - high level scripting language that is used with Apache Hadoop. Pig excels at describing data analysis problems as data flows. Pig is complete in that you can do all the required data manipulations in Apache Hadoop with Pig. In addition through the User Defined Functions(UDF) facility in Pig you can have Pig invoke code in many languages like JRuby, Jython and Java. Conversely you can execute Pig scripts in other languages. The result is that you can use Pig as a component to build larger and more complex applications that tackle real business problems. A good example of a Pig application is the ETL transaction model that describes how a process will extract data from a source, transform it according to a rule set and then load it into a datastore. Pig can ingest data from files, streams or other sources using the User Defined Functions(UDF). Once it has the data it can perform select, iteration, and other transforms over the data. Again the UDF feature allows passing the data to more complex algorithms for the transform. Finally Pig can store the results into the Hadoop Data File System. Pig scripts are translated into a series of MapReduce jobs that are run on the Apache Hadoop cluster. As part of the translation the Pig interpreter does perform optimizations to speed execution on Apache Hadoop.  

**Presto** (Facebook) [SQL][INTERACTIVE SQL][ANSI SQL] - Facebook’s version of Cloudera’s Impala SQL querying engine or what Hortonworks is working on with Stinger, but custom-fit for fast performance at Facebook scale. New open source distributed SQL query engine for interactive analytics on big data. This talk we will discuss the motivation and main design points behind developing Presto at Facebook, and delve into the architecture details. In addition to querying over Hive/HDFS, Presto has an extensible plugin model that enables query capability over other data stores such as Hbase, etc.  

**Samza** (Apache) [STREAM PROCESSING] - Apache Samza is a stream processor LinkedIn recently open-sourced. Near-realtime, asynchronous computational framework for stream processing.  

**SOLR** [SEARCH] - popular, blazing fast open source enterprise search platform from the Apache Lucene project.  Its major features include powerful full-text search, hit highlighting, faceted search, near real-time indexing, dynamic clustering, database integration, rich document handling, and geospatial search.  

**Shark** [SQL] - stopped development, replaced with Spark SQL  

**Spark** [MR ENGINE][PROCESSING ENGINE] - Apache Spark is an execution engine that broadens the type of computing workloads Hadoop can handle, while also tuning the performance of the big data framework. Spark is a fast and powerful engine for processing Hadoop data. It runs in Hadoop clusters through Hadoop YARN or Spark's standalone mode, and it can process data in HDFS, HBase, Cassandra, Hive, and any Hadoop InputFormat. It is designed to perform both general data processing (similar to MapReduce) and new workloads like streaming, interactive queries, and machine learning.  

**Spark Streaming** [STREAM PROCESSING] - is a component of Spark that provides highly scalable, fault-tolerant streaming processing.  Extension of the core Spark API that allows enables scalable, high-throughput, fault-tolerant stream processing of live data streams. Data can be ingested from many sources like Kafka, Flume, Twitter, ZeroMQ, Kinesis or plain old TCP sockets and be processed using complex algorithms expressed with high-level functions like map, reduce, join and window. Finally, processed data can be pushed out to filesystems, databases, and live dashboards. In fact, you can apply Spark’s machine learning algorithms, and graph processing algorithms on data streams.  

**Spark SQL** [INTERACTIVE SQL][SQL] - Spark SQL allows relational queries expressed in SQL, HiveQL, or Scala to be executed using Spark. At the core of this component is a new type of RDD, SchemaRDD. SchemaRDDs are composed of Row objects, along with a schema that describes the data types of each column in the row. A SchemaRDD is similar to a table in a traditional relational database. A SchemaRDD can be created from an existing RDD, a Parquet file, a JSON dataset, or by running HiveQL against data stored in Apache Hive.  

**Sqoop** (Apache) - tool designed for efficiently transferring bulk data between Apache Hadoop and structured datastores such as relational databases. It offers two-way replication with both snapshots and incremental updates.  

**Stinger** (Hortonworks) [INTERACTIVE SQL][MR] - optimized Hive. Generates more optimized jobs.  

**Storm** [STREAM PROCESSING] - distributed real-time computation system for processing fast, large streams of data. Storm adds reliable real-time data processing capabilities to Apache Hadoop® 2.x. Storm in Hadoop helps capture new business opportunities with low-latency dashboards, security alerts, and operational enhancements integrated with other applications running in their Hadoop cluster.  

**Tableau** [VISUALIZATION][ANALYTIC][SQL] - set. Tableau can directly access the data in the Hadoop Platform, as well as the data in traditional analytic databases, and can combine them in a single view using a capability known as “data blending.” Tableau can then explore and visualize the blended data, providing valuable business. (BA tool, reports, UI web, UI client, JDBC driver)  

**Tachyon** [DFS] - memory-centric distributed file system enabling reliable file sharing at memory-speed across cluster frameworks, such as Spark and MapReduce. It achieves high performance by leveraging lineage information and using memory aggressively. Tachyon caches working set files in memory, thereby avoiding going to disk to load datasets that are frequently read. This enables different jobs/queries and frameworks to access cached files at memory speed. Tachyon is Hadoop compatible. Existing Spark and MapReduce programs can run on top of it without any code change. The project is open source (Apache License 2.0) and is deployed at multiple companies. It has more than 40 contributors from over 15 institutions, including Yahoo, Intel, and Redhat.  

**Tez** (Apache) [MR ENGINE][PROCESSING ENGINE] - is an extensible framework for building YARN based, high performance batch and interactive data processing applications in Hadoop that need to handle TB to PB scale datasets. It allows projects in the Hadoop ecosystem, such as Apache Hive and Apache Pig, as well as 3rd-party software vendors to express fit-to-purpose data processing applications in a way that meets their unique demands for fast response times and extreme throughput at petabyte scale. Apache Tez  provides a developer API and framework to write native YARN applications that bridge the spectrum of interactive and batch workloads  

**YARN** [RESOURCEMANAGER] - YARN is the prerequisite for Enterprise Hadoop, providing resource management and a central platform to deliver consistent operations, security, and data governance tools across Hadoop clusters.   

**ZooKeeper** [CLUSTER COORDINATION] - coordination service and used by Storm, Hadoop, HBase, ElasticSearch and other distributed computing frameworks. Service for coordinating processes of distributed applications. Historically distributed processes are coordinated using group messaging, shared registers, or distributed lock services. ZooKeeper incorporates elements from all these servers, but incorporates them into a replicated centralized service. The interface exposed by ZooKeeper incorporates the wait-free aspects of group messaging and shared registers with an eventing mechanism similar to those of locking services to provide a simple, yet powerful coordination service  

### Who is who ###

Well, as Architect I need to know which technologies and frameworks are compatible and pros and cons of every solution.

So, I've created mind-map which is [**>> available by this link for full-screen review <<**] (https://www.mindmeister.com/547743421/bigdata).  

<iframe width="1000" height="1000" frameborder="0" src="https://www.mindmeister.com/maps/public_map_shell/547743421/bigdata?width=1000&height=1000&z=auto" scrolling="no" style="overflow:hidden">Your browser is not able to display frames. Please visit the <a rel="nofollow" href="https://www.mindmeister.com/547743421/bigdata" target="_blank">mind map: BigData</a> on <a rel="nofollow" href="http://www.mindmeister.com" target="_blank">Mind Mapping - MindMeister</a>.</iframe>

If you have an account on MindMester you'll be able to export or clone this schema.

Also you can download [big picture](/assets/posts/2015-05-31-who-is-who-in-bigdata/bigdata.png) or [source file for MindMeister](/assets/posts/2015-05-31-who-is-who-in-bigdata/bigdata.mind)

Enjoy!


