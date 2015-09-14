---
layout: post
title:  "Notes on NoSQL : Cassandra"
date:   2015-09-13 16:00:00
categories: NoSQL Architecture
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Principal Software Engineer and Grid Architect in San Francisco, USA. You can subscribe to my new posts in my <a href="http://sotnikdv.github.io">personal blog</a> or find me in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
editable: false
---

In this article we will overview one of the NoSQL databases, **Cassandra**. We will review architecture, key strengths and weakness and usage experience, if available.

This is the one of the few articles in "Notes on NoSQL" sequence  
[Notes on NoSQL : Basics](/nosql/architecture/2015/09/12/notes-on-nosql-basics.html)  
[Notes on NoSQL : MongoDB](/nosql/architecture/2015/09/13/notes-on-nosql-mongodb.html)  
**[Notes on NoSQL : Cassandra](/nosql/architecture/2015/09/13/notes-on-nosql-cassandra.html)**  
[Notes on NoSQL : HBase](/nosql/architecture/2015/09/13/notes-on-nosql-hbase.html)  
[Notes on NoSQL : Redis](/nosql/architecture/2015/09/13/notes-on-nosql-redis.html)  
[Notes on NoSQL : Aerospike](/nosql/architecture/2015/09/13/notes-on-nosql-aerospike.html)  

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
  <h5>This post is a summary, so you have to read all references</h5>
  <p>
    This post is a brief summary and not intended to explain all related terms, but provides references instead. Please read all related materials. If some references missed - please improve the article.
  </p>
</div>

* TOC
{:toc}

**Cassandra** is the free and Open-source database, written in Java. Datastax make business mostly on a support.

Used by many clients, Netflix shown benchmarks with "[Over a million writes per second on AWS](http://techblog.netflix.com/2011/11/benchmarking-cassandra-scalability-on.html)" (see also [presentation](http://www.slideshare.net/adrianco/cassandra-performance-on-aws))

**Wide-column** database (group of columns associated with the single key) based on [Log-Structured Merge-Tree](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.44.2782). [Datafiles are organized as SSTable](http://teddyma.gitbooks.io/learncassandra/content/model/where_is_data_stored.html) (SSTable is a simple abstraction to efficiently store large numbers of key-value pairs while optimizing for high throughput, sequential read/write workloads, so key cache, row cache and indexes are used).

Starting CQL3 require columns to be declared before used, so Cassandra is not anymore "schemaless", however [using a Map on Cassandra 2.1 we can store unstructured data](http://stackoverflow.com/questions/25098451/cql3-each-row-to-have-its-own-schema).

**JSON** datatype is not supported as schemaless and [feature request was rejected](https://issues.apache.org/jira/browse/CASSANDRA-6833). 

Nevertheless, [starting v 2.2](https://issues.apache.org/jira/browse/CASSANDRA-7970) JSON is [supported as input/output](http://www.datastax.com/dev/blog/whats-new-in-cassandra-2-2-json-support) format in [CQL](http://cassandra.apache.org/doc/cql3/CQL-2.2.html#insertJson). This may lead to conclusion, that since 01/Apr/15 Cassandra become **document-oriented**-like DB with **fixed document schema** (need to be confirmed).

# Architecture

Decentralized distributed read-anywhere-write-anywhere peer-to-peer based architecture. One or more of the nodes in a cluster act as replicas for a given piece of data. Number of replicas is configured.

Behaviour of the cluster and consistency controlled by 4 parameters:

* cluster size

* replication factor (number of replicas)

* read consistency (1,2,3,quorum,all)

* write consistency (1,2,3,quorum,all)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_4.png)

Note: coordinator node sends a write request to all replicas regardless of Consistency Level. Confirmation from CL-number of replicas required to confirm write to a client.

Read flow in this schema is simple:

* coordinator node sends direct read request to CL number of fastest replicas (Dynamic Snitch), 1 request for full read, CL-1 for digest read

* if there is a match - it is returned to client

* if there is mismatch, coordinator node sends direct full read request to CL number of replicas and most recent copy returned to client. After returning the most recent value, Cassandra performs a read repair in the background to update the stale values.

See also [presentation from Grid Dynamics](http://www.slideshare.net/Open-IT/cassandra-meetup-april-2014).

# Data consistency model

**Strong** to **eventual** consistency depending on configuration ([tunable consistency](http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html)), more strong consistency will affect latency.

Cluster is **strong consistent** until number of nodes, required to read + nodes, required to write > replication factor. So QUORUM for read/write will guarantee **strong consistency**.

Cassandra chose **not** to implement vector clocks for performance reasons. Vclocks (typically) require a read before each write. By using last-write-wins in all cases, and ignoring the causality graph, Cassandra can cut the number of round trips required for a write and obtain a significant speedup. [The downside is that there is no safe way to modify a Cassandra cell, last write wins](https://aphyr.com/posts/294-call-me-maybe-cassandra/).

See also [Cassandra Consistency Features](https://quabase.sei.cmu.edu/mediawiki/index.php/Cassandra_Consistency_Features).

# ACID guarantees

Accordingly to [documentation](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_about_transactions_c.html) Cassandra does not use RDBMS ACID transactions with rollback or locking mechanisms, but instead offers atomic, isolated, and [durable transactions](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_ltwt_transaction_c.html) with eventual/tunable consistency that lets the user decide how strong or eventual they want each transaction’s consistency to be.

As a non-relational database, Cassandra does not support joins or foreign keys, and consequently does not offer consistency in the ACID sense. For example, when moving money from account A to B the total in the accounts does not change. Cassandra supports atomicity and isolation **at the row-level**, but trades transactional isolation and atomicity for high availability and fast write performance. Cassandra writes are durable.

[Distributed lightweight transactions](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_ltwt_transaction_c.html) are based on Paxos protocol.

So ACID guarantees of Cassandra are next

**Atomicity**: write is atomic at row-level; doesn’t rollback in case of fail on some replicas

**Consistency**: tunable with CL requirement (CP vs AP)

**Isolation**: row-level

**Durability**: durable, but persistence is asynchronous by default (tunable)

Also lightweight transactions available since Cassandra 2.0 for INSERT and UPDATE

# Replication schema

Cassandra stores few copies (replicas) of every partition, number of copies depends on replication factor. All replicas are equally important; there is no primary or master replica. 

Usage of replicas described in architecture section.

[While replication two strategies available](http://www.slideshare.net/DataStax/understanding-data-partitioning-and-replication-in-apache-cassandra): **simple strategy** (without considering rack or datacenter location) and **network topology** (more control over where replica rows are placed)

Also there is a [hinted handoff for cases, when replicas are offline, but write consistency met](http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_about_hh_c.html). Hinted handoff optimizes the cluster consistency process and anti-entropy when a replica-owning node is not available, due to network issues or other problems, to accept a replica from a successful write operation. Hinted handoff is not a process that guarantees successful write operations, [except when a client application uses a consistency level of ANY](http://www.slideshare.net/DataStax/understanding-data-consistency-in-apache-cassandra)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_5.png)

# Sharding Schema

Data is partitioned in Cassandra across participating nodes based on column family row key. 

Two basic strategies: 

* Random partitioning (default and recommended). MD5 hash from every column family row key used

* Ordered partitioning (ByteOrdered and OrderPreserving). Stores column family row keys in sorted order. Not recommended because of this limitation and because globally ordering all your partitions generates hot-spots.

# CAP-theorem

Cassandra is typically classified as an AP system, meaning that availability and partition tolerance are generally considered to be more important than consistency in Cassandra. But Cassandra can be tuned with replication factor and consistency level to also meet CP.

# Split-brain and recovery from a split brain

Cassandra may be vulnerable to split-brain if configured as AP-system.

Recovery from split-brain include schema and data merge. 

Data merged by "last-wins" principle. Schema disagreement [may require manual recovery](http://wiki.apache.org/cassandra/LiveSchemaUpdates)   

# Rack-aware replication and cross-datacenter replication

[Accordingly to documentation](http://docs.datastax.com/en/cassandra/2.0/cassandra/architecture/architectureDataDistributeReplication_c.html), Cassandra supports rack-aware and datacenter-aware replication (multi-datacenter replication). "Last wins" merge policy used.

From experience, multi-datacenter replication was used under heavy load for years and was pretty stable.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_6.png)

# Fault-tolerance

High-availability and fault tolerance is built on replication and [peer-to-peer architecture to survive replica fail](http://www.datastax.com/dev/blog/how-cassandra-deals-with-replica-failure)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_7.png)

Fault-tolerance (how many nodes may be lost) depends on 

* replication factor

* consistency level used for read and writes

* network topology settings

In general, Cassandra will survive loss of (REPLICATION FACTOR - CONSISTENCY LEVEL NODES COUNT) nodes

# Dynamic scalability

[Cassandra is dynamically scalable](http://docs.datastax.com/en/cassandra/2.0/cassandra/operations/opsAddingRemovingNodeTOC.html) (add/remove nodes and datacenters). [Network topology change may require downtime of specific node or full cluster repair.](http://docs.datastax.com/en/cassandra/2.0/cassandra/operations/opsMoveNodeRack.html)

Cassandra automatically rebalance data if new node added, but d[oes not automatically remove data from nodes that lose part of their partition range to a newly added node](http://docs.datastax.com/en/cassandra/2.0/cassandra/tools/toolsCleanup.html) and manual cleanup required.

# Persistency

Cassandra is using [due to log-structured merge-tree](https://github.com/wiredtiger/wiredtiger/wiki/LSMTrees) for high write throughput, so Cassandra is [mostly write-oriented](https://github.com/wiredtiger/wiredtiger/wiki/Btree-vs-LSM). In the same time, read throughput and latency are good enough.

Cassandra persist data accordingly to durability settings, but synchronous persistency is slow. 

In default mode write confirmed when node added change to memory and distributed over the cluster, commit log will be dumped to disk **by timeout**. Cassandra writes are first [written to the CommitLog](https://wiki.apache.org/cassandra/Durability), and then to a [per-ColumnFamily structure called a Memtable](http://codrspace.com/b441berith/cassandra-sstable-memtable-inside/). When a Memtable is full, [it is written to disk as an SSTable](https://wiki.apache.org/cassandra/MemtableSSTable). Also there is [compaction process](http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_write_path_c.html).

## Write parh

When a [write occurs](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html), Cassandra stores the data in a structure in memory, the memtable, and also appends writes to the commit log on disk (**dumped asynchronously by default**), providing configurable durability. The commit log receives every write made to a Cassandra node, and these durable writes survive permanently even after power failure (but depends on settings, asynchronous by default). 

The memtable is a **write-back cache** of data partitions that Cassandra looks up by key. The memtable stores writes until reaching a limit (configurable), and then is flushed. replay time. 

To flush the data, Cassandra sorts memtables by token and then writes the data to disk sequentially.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_8.png)

## Read path

When a [read request for a row comes in to a node](http://docs.datastax.com/en/cassandra/1.2/cassandra/dml/dml_about_read_path_c.html), the row must be combined from all SSTables on that node that contain columns from the row in question, as well as from any unflushed memtables, to produce the requested data.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_9.png)

# Query Language, Entry Processor, User Defined Functions and Bulk Writes

CQL query language available without analytical functionality.

There is no Entry Processor in this DB, but there are triggers. In the same time, 1 year ago Datastax engineers unofficially didn’t recommended to not use them due to feature is new and maybe unstable.

# Indexes (local and global)

[Secondary indexes](http://docs.datastax.com/en//cql/3.1/cql/ddl/ddl_when_use_index_c.html) [are supported](https://issues.apache.org/jira/browse/CASSANDRA-3680). Indexes are local (each node only indexes data that it holds locally) and distributed over the cluster, so for search request will be sent to multiple nodes in parallel. 

Also this means that you must have CL nodes available for all token ranges in the cluster in order to complete a query regardless of cluster size (for example, with RF = 3, when two out of three consecutive nodes in the ring are unavailable, all secondary index queries at CL = QUORUM will fail, however secondary index queries at CL = ONE will succeed)

The row and index updates are one, atomic operation.

# Traffic encryption and access control

[Node-to-node](http://docs.datastax.com/en/cassandra/1.2/cassandra/security/secureSSLNodeToNode_t.html) and [client-to-node](http://docs.datastax.com/en/cassandra/1.2/cassandra/security/secureSSLClientToNode_t.html) encryption of data transferred between nodes, including gossip communication available with using SSL

[Internal authentication](http://docs.datastax.com/en/cassandra/1.2/cassandra/security/secureInternalAuthenticationTOC.html) and [internal authorization](http://docs.datastax.com/en/cassandra/1.2/cassandra/security/secureInternalAuthorizationTOC.html) with object permissions is available.

[LDAP and Kerberos integration](http://docs.datastax.com/en/datastax_enterprise/4.6/datastax_enterprise/sec/secTOC.html) is available with [DataStax Enterprise](http://docs.datastax.com/en/datastax_enterprise/4.6/datastax_enterprise/newFeatures.html).

# Caching

Starting Cassandra 2.0 [supports off-heap storages](http://www.datastax.com/dev/blog/off-heap-memtables-in-Cassandra-2-1) for [bloom filters, compression offsets map and partition summary](http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_off_heap_c.html). Also [row cache may be placed off-heap](http://docs.datastax.com/en/cassandra/2.0/cassandra/operations/ops_configuring_caches_c.html), but this will lead to de-serialization overhead. Key cache is on-heap.

# Business model and commercial support

DataStax Enterprise is [available by subscription](http://www.datastax.com/what-we-offer/products-services/production).

# Usage experience

Cassandra was used as storage (16 nodes, cross-datacenter replication), our experience is next:

* Good horizontal scalability with automatic sharding.

* Stable work on 2000 - 3000 tps with stable master-master cross-datacenter replication

* CQL is not so advanced as SQL, no joins, group by, select...where on arbitrary columns and analytical functions

* No table statistic, so select count is expensive (full scan), so if we need statistic - need to support statistic table.

* Cassandra may be affected by GC sometimes, so if SLA, for example, is strongly >99% of <10ms for request time, then Cassandra should not be used. 

* There is no performance gain from key and row caches if dataset fits memory (this is obvious)

* Query engine may be memory intensive on complicated queries (select count limit 50 000 will lead to allocating of result set for 50 000 rows, which is unnecessary)

* Two years of reliable work in production without significant issues

* Stable cross-datacenter replication

* Used as a Catalog storage instead of Coherence to simplify deployment architecture by eliminating primary-secondary schema for fast recover without Coherence persistency usage. Cassandra is very fast when dataset is in memory (1-2 msec read).

* By our experience, Cassandra is good as reliable records storage with high-throughput, horizontal scalability and reliable cross-datacenter replication, if latency > 20-30ms but less than 50ms average acceptable

# Additional Materials, required to read

[Schema in Cassandra 1.1 and composite columns](http://www.datastax.com/dev/blog/schema-in-cassandra-1-1)

[Common Cassandra Data Modelling Traps](https://www.instaclustr.com/common-cassandra-data-modelling-traps/)

[How does the Log-Structured-Merge-Tree work?](http://www.quora.com/How-does-the-Log-Structured-Merge-Tree-work)

[A Comparison of Fractal Trees to Log-Structured Merge (LSM) Trees](http://insideanalysis.com/wp-content/uploads/2014/08/Tokutek_lsm-vs-fractal.pdf)

[bLSM:∗ A General Purpose Log Structured Merge Tree. Yahoo Research](http://www.eecs.harvard.edu/~margo/cs165/papers/gp-lsm.pdf)

[The Log-Structured Merge-Tree (LSM-Tree)](http://www.cs.umb.edu/~poneil/lsmtree.pdf)

[A Comparison of Fractal Trees to Log-Structured Merge (LSM) Trees](http://form.percona.com/rs/percona/images/wp_lsm_vs_fractal.pdf)

[B-tree](https://en.wikipedia.org/wiki/B-tree)

[Bigtable: A Distributed Storage System for Structured Data](http://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf)

[SSTable and Log Structured Storage: LevelDB](https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/)

[How Cassandra stores indexes](http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_index_internals.html)

[Cassandra background-and-architecture](http://www.slideshare.net/yellow7/cassandra-backgroundandarchitecture)

[GridDynamics Cooking Cassandra](http://www.slideshare.net/Open-IT/cassandra-meetup-april-2014) presentation

[Using Apache Cassandra: What is this thing, and how do I use it?](http://www.slideshare.net/jeremiahdjordan/cassandra-intro-27171544)

[NoSQL Essentials: Cassandra](http://www.slideshare.net/frodriguezolivera/nosql-essentials-cassandra-15625311)
