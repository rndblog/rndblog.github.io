---
layout: post
title:  "Notes on NoSQL : Apache HBase, database on the top of Hadoop"
date:   2015-09-13 17:00:00
update: 2015-09-14 12:55:00
categories: NoSQL Architecture
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Principal Software Engineer and Grid Architect in San Francisco, USA. You can subscribe to my new posts in my <a href="http://sotnikdv.github.io">personal blog</a> or find me in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
editable: false
---

**HBase** is the key-value NoSQL DB build on top of HDFS (distributed file system). Written on Java, distributed with the most of Hadoop distributions. HBase is also **wide-column** and similar file format and architecture as Cassandra. HBase supports a "bytes-in/bytes-out".

This is the one of the few articles in "Notes on NoSQL" sequence  
[Notes on NoSQL : Basics and key differences between relational and NoSQL databases](/nosql/architecture/2015/09/13/notes-on-nosql-basics.html)  
[Notes on NoSQL : MongoDB, unfulfilled hopes and possible bright future](/nosql/architecture/2015/09/13/notes-on-nosql-mongodb.html)  
[Notes on NoSQL : Years in production with Apache Cassandra](/nosql/architecture/2015/09/13/notes-on-nosql-cassandra.html)  
**[Notes on NoSQL : Apache HBase, database on the top of Hadoop](/nosql/architecture/2015/09/13/notes-on-nosql-hbase.html)**  
[Notes on NoSQL : Redis, pretty simple and jet fast cache](/nosql/architecture/2015/09/13/notes-on-nosql-redis.html)  
[Notes on NoSQL : Aerospike and risqué promises](/nosql/architecture/2015/09/13/notes-on-nosql-aerospike.html)  

<div class="note unreleased">
  <h5>Early Preview version</h5>
  <p>
    This post is the Early Preview which is not ready to be distributed and not recommended to be used. A lot of mistypes, inaccuracies and mistakes inside which are not fixed yet.
  </p>
</div>

<div class="note info">
  <h5>No change requests will be accepted until release</h5>
  <p>
    This post is not published officially yet, so please wait for official release before submitting any changes. Any pull requests will be rejected because article will be heavily re-worked very soon.
  </p>
</div>

<div class="note info">
  <h5>This post is the summary (compilation) from many sources, you have to read all references</h5>
  <p>
    Post is not intended to explain all related terms or re-write all information from origins. So, please consider this post as the "navigation map" and read all references in the text body and in the end of the post.
  </p>
</div>

<div class="note info">
  <h5>A special thanks to <a href="http://griddynamics.com">Grid Dynamics Inc.</a> and colleagues from the author</h5>
  <p>
    This post was initially written as notes in my personal time, in the period when I worked on related topic for Grid Dynamics Inc. <br/>I'd like to thanks to the company and colleagues for the great support and knowledge shared. Personal thanks to <a href="https://www.linkedin.com/in/stryuber">Sergey Tryuber</a>
  </p>
</div>


* TOC
{:toc}

# Architecture

Data in HBase is stored in Tables and these Tables are stored in Regions. When a Table becomes too big, the Table is partitioned into multiple Regions. These Regions are assigned to Region Servers across the cluster. Each Region Server hosts roughly the same number of Regions.

Query will be routed to corresponding region server automatically by client driver.

Physically, [HBase is composed of three types of servers in a master-slave type of architecture](https://www.mapr.com/blog/in-depth-look-hbase-architecture): 

* Region servers serve data for reads and writes. When accessing data, clients communicate with HBase RegionServers directly. Region assignment, DDL (create, delete tables) operations are handled by the HBase Master process. Zookeeper, which is part of HDFS, maintains a live cluster state.

* The Hadoop DataNode stores the data that the Region Server is managing. All HBase data is stored in HDFS files. Region Servers are collocated with the HDFS DataNodes, which enable data locality (putting the data close to where it is needed) for the data served by the RegionServers. HBase data is local when it is written, but when a region is moved, it is not local until compaction.

* The NameNode maintains metadata information for all the physical data blocks that comprise the files.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_10.png)

[The HMaster in the HBase is responsible for](http://netwovenblogs.com/2013/10/10/hbase-overview-of-architecture-and-data-model/)

* Performing Administration

* Managing and Monitoring the Cluster

* Assigning Regions to the Region Servers

* Controlling the Load Balancing and Failover

* DDL

HRegionServer perform the following work:

* Hosting and managing Regions

* Splitting the Regions automatically

* Handling the read/write requests

* Communicating with the Clients directly

Each Region Server contains a Write-Ahead Log (called HLog) and multiple Regions. Each Region in turn is made up of a MemStore and multiple StoreFiles (HFile). The data lives in these StoreFiles in the form of Column Families (explained below). The MemStore holds in-memory modifications to the Store (data).

The mapping of Regions to Region Server is kept in a system table called .META. When trying to read or write data from HBase, the clients read the required Region information from the .META table and directly communicate with the appropriate Region Server. Each Region is identified by the start key (inclusive) and the end key (exclusive) 

Map-Reduce is NOT used, so statements about big latency due to MR is a myth, HBase can be used without YARN at all. Nevertheless, HDFH usage may impose additional latency.

More information on architecture [available here](https://www.mapr.com/blog/in-depth-look-hbase-architecture).

# Data consistency model

Due to Region Master HBAse support **strong consistency** at the level of a single row.

[In fact HBase ](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.3/bk_system-admin-guide/content/sysadminguides_ha-HBase-data-consistency.html)[guarantees timeline consistency](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.3/bk_system-admin-guide/content/sysadminguides_ha-HBase-data-consistency.html) for all data served from Region Servers in secondary mode, meaning all HBase clients see the same data in the same order, but that data may be slightly stale. Only the primary Region Server is guaranteed to have the latest data. 

Timeline consistency simplifies the programming logic for complex HBase queries and provides lower latency than quorum-based consistency

# ACID guarantees

[HBase supports ACID in limited ways](http://hadoop-hbase.blogspot.com/2012/03/acid-in-hbase.html), namely Puts to the same row provide all ACID guarantees. ([HBASE-3584](https://issues.apache.org/jira/browse/HBASE-3584) adds multi op transactions and [HBASE-5229](https://issues.apache.org/jira/browse/HBASE-5229) adds multi row transactions, but the principle remains the same)

# Replication schema

[Replication is based on HDFS ](http://hortonworks.com/blog/introduction-to-hbase-mean-time-to-recover-mttr/)functionality.

Data written in HDFS is replicated on several nodes:

* HBase writes the data in HFiles, stored in HDFS. HDFS replicates the blocks of these files, by default 3 times.

* HBase uses a commit log (or Write-Ahead-Log, WAL), and this commit log is as well written in HDFS, and as well replicated, again 3 times by default.

# Sharding Schema

Sharding is configurable by splitting on regions. Initially, [HBase creates only one region per table](http://hortonworks.com/blog/apache-hbase-region-splitting-and-merging/). Once a region gets to a certain limit, it is automatically split into two regions. HBase also enables clients to force split an online table from the client side.

Master includes load balancing between origin servers and automatic re-balancing of shards. 

Master keep information about table to region split and region to origin mapping (Zookeeper used). 

Client download meta-information from master until region is missed, then will re-download (when origin server is unavailable or region unavailable on origin server)

# CAP-theorem

HBase is CP-system (HDFS).

# Split-brain and recovery from a split brain

Not affected to split-brain because it’s CP.

# Rack-aware replication and cross-datacenter replication

Replication in [HDFS is rack-aware](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html).

Cluster to cluster replication can be defined at the column-family level, works in the background, and keeps all edits in sync between clusters in the replication chain. Replicationis based on distribution of WAL.

Cluster-to-cluster replication has three modes: master-slave, master-master, and cyclic.

# Fault-tolerance

HBase fault tolerance is based on:

* HA of multiple master nodes

* HA of HDFS (HA of HDFS name nodes based on Zookeeper cluster usage)

* HDFS replication

So, in general, HBase HA is relying on Hadoop HA and data replication.

[Steps in the failure detection and recovery process](http://hortonworks.com/blog/introduction-to-hbase-mean-time-to-recover-mttr/):

* Identifying that a node is down: a node can cease to respond simply because it is overloaded or as well because it is dead.

* Recovering the writes in progress: that’s reading the commit log and recovering the edits that were not flushed.

* Reassigning the regions: the region server was previously handling a set of regions. This set must be reallocated to other region servers, depending on their respective workload.

Until the detection and recovery steps have happened, the client is blocked – a single major pain point! Expediting the process, so that clients see less downtime of their data while preserving data consistency is what MTTR (mean time to recover) is all about.

There are no global failures in HBase: if a region server fails, all the other regions are still available. For a given data-subset, the MTTR was often considered as around **ten minutes**. 

This rule of thumb was actually coming from a common case where the recovery was taking time because it was trying to use replicas on a dead datanode. Ten minutes would be the time taken by HDFS to declare a node as dead. With the new stale mode in HDFS, it’s not the case anymore, and the recovery is now bounded by HBase alone. Most cases will take less than **2 minutes between** the actual failure and the data being available again in another region server.

[In the same time Hortonworks declared HBase Read HA in HDP 2.2](http://hortonworks.com/blog/apache-hbase-high-availability-next-level/)

# Dynamic scalability

[Adding of a new Regions Server](http://hbase.apache.org/book.html#adding.new.node) doesn’t require downtime, but [Region Server decommission](http://hbase.apache.org/book.html#decommission) may lead to Region temporary unavailability 

# Persistence

Persistence is based on HDFS and similar to Cassandra (write-ahead-log with memstore which is flushed asynchronously)

Good overview of the read-write paths can be found in this presentation: [Storage Systems for big data - HDFS, HBase, and intro to KV Store - Redis ](http://www.slideshare.net/sameertiwari33/storage-for-big-data-30957881)

The main concern is a durability of a WAL log, need to confirm that [WAL flush](http://blog.cloudera.com/blog/2012/06/hbase-write-path/) is synchronous.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_11.png)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_12.png)

# Query Language, Entry Processor, User Defined Functions and Bulk Writes

HBase have kind of EntryProcessors (CoProcessors) which behaves like triggers, nevertheless coprocessors are not designed to be used by end users of HBase, but by HBase developers who need to add specialized functionality to HBase. 

No Query Language for HBase, but there is 3rd party project, Phoenix, QL for HBase. 

Also [Phoenix project provides set of indexes, including global indexes](https://phoenix.apache.org/secondary_indexing.html) for read heavy, low writes cases

Bulk writes are supported.

# Indexes (local and global)

As well as in Cassandra, [indexes are local (region-level)](http://www.z2-environment.net/blog/2014/01/secondary-indexes-in-hbase/) and integrated with HBase’s Write-Ahead-Log (WAL), indexes are written directly and atomically for data within the region server local regions to the region server.

So, every region server holds indexes for its data. This approach is implemented by HIndex and IHBase. This may affects scalability for broad queries. 

For global indexes (covering indexes) see Phoenix

# Traffic encryption and access control

Traffic encryption, data encryption and access control based on Hadoop security (SSL, LDAP and RBAC)

# Caching schema

In the default configuration, [HBase uses a single on-heap cache](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/admin_hbase_blockcache_configure.html). Off-heap [BucketCache](http://hortonworks.com/blog/hbase-blockcache-101/) can be configured and will be used for [Bloom filters](https://en.wikipedia.org/wiki/Bloom_filter), indexes and cache data blocks.

# Business model and commercial support

HBase is the part of major Hadoop distributions and supported by distribution vendors

# Usage experience

HBase was used on one of the projects to store and analyze clickstream few years, results are next:

* stable enough for production usage

* CLI interface is not rich and just get\put    

# Recommendations

HBase is good to use with Hadoop, utilizing existing Hadoop deployment. HBase will provide random access instead of sequential to Hadoop plus HBase uses existing Hadoop architecture

HBase better for Hadoop then remote storage (Cassandra etc.) because you don’t need to make long network call to remote system and transfer data. In case of HBase usage, local map task may use region stored on the same node. Also there is extension for MR to read/parse HBase files (HFileInputFormat), so analytic over HBAse is extremely fast.

Usage of HBase as a pure NoSQL solution raises two concerns:

* pipeline of request processing is simply longer than in Cassandra and requires more work, see read path. Nevertheless [was a posts which said that HBase may be fast as Cassandra](http://www.slideshare.net/HBaseCon/features-session-5). 

* HBase relies on Hadoop architecture and deployment, so it’s a cheap and simple solution if used with Hadoop, but expensive (HDFS, Zookeeper) if used as pure NoSQL storage.

# Additional Materials, required to read

[How does the Log-Structured-Merge-Tree work?](http://www.quora.com/How-does-the-Log-Structured-Merge-Tree-work)

[A Comparison of Fractal Trees to Log-Structured Merge (LSM) Trees](http://insideanalysis.com/wp-content/uploads/2014/08/Tokutek_lsm-vs-fractal.pdf)

[bLSM:∗ A General Purpose Log Structured Merge Tree. Yahoo Research](http://www.eecs.harvard.edu/~margo/cs165/papers/gp-lsm.pdf)

[The Log-Structured Merge-Tree (LSM-Tree)](http://www.cs.umb.edu/~poneil/lsmtree.pdf)

[A Comparison of Fractal Trees to Log-Structured Merge (LSM) Trees](http://form.percona.com/rs/percona/images/wp_lsm_vs_fractal.pdf)

[B-tree](https://en.wikipedia.org/wiki/B-tree)

[SSTable and Log Structured Storage: LevelDB](https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/)

[Secondary indexes in HBase Phoenix](https://github.com/forcedotcom/phoenix/wiki/Secondary-Indexing)

[HBase : Secondary Indexes and Alternate Query Paths](http://hbase.apache.org/book.html#secondary.indexes)

[Apache HBase Reference Guide](http://hbase.apache.org/book.html)

[Secondary Indexes in HBase](http://www.z2-environment.net/blog/2014/01/secondary-indexes-in-hbase/#gti_refs)
