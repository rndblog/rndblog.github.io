---
layout: post
title:  "Notes on NoSQL : Aerospike"
date:   2015-09-13 19:00:00
categories: NoSQL Architecture
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Principal Software Engineer and Grid Architect in San Francisco, USA. You can subscribe to my new posts in my <a href="http://sotnikdv.github.io">personal blog</a> or find me in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
editable: false
---

In this article we will overview one of the NoSQL databases, **Aerospike**. We will review architecture, key strengths and weakness and usage experience, if available.

This is the one of the few articles in "Notes on NoSQL" sequence  
[Notes on NoSQL : Basics](/nosql/architecture/2015/09/12/notes-on-nosql-basics.html)  
[Notes on NoSQL : MongoDB](/nosql/architecture/2015/09/13/notes-on-nosql-mongodb.html)  
[Notes on NoSQL : Cassandra](/nosql/architecture/2015/09/13/notes-on-nosql-cassandra.html)  
[Notes on NoSQL : HBase](/nosql/architecture/2015/09/13/notes-on-nosql-hbase.html)  
[Notes on NoSQL : Redis](/nosql/architecture/2015/09/13/notes-on-nosql-redis.html)  
**[Notes on NoSQL : Aerospike](/nosql/architecture/2015/09/13/notes-on-nosql-aerospike.html)**  

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

Aerospike is the flash-optimized in-memory [wide-column database with schema-less data model](http://www.aerospike.com/docs/guide/kvs.html) with [strong typization](http://www.aerospike.com/docs/guide/data-types.html) and [large collections](http://www.aerospike.com/docs/guide/ldt.html). Written in C.

JSON is not supported, [the only way to store JSON is a string or decomposition to List/Map](https://github.com/aerospike/complex-data-types).

# Architecture

Data is organized into policy containers called namespaces, semantically similar to databases in an RDBMS system

Aerospike divided onto

* [Client Layer](http://www.aerospike.com/docs/architecture/clients.html). Implements the Aerospike API, the client-server protocol and talks directly to the cluster; Tracks nodes and knows where data is stored; Transparently sends requests directly to the node with the data and re-tries or re-routes requests as needed

* Distribution Layer.  Responsible for cluster management (failover, replication, cross-datacenter synchronization), transaction management and duplicate resolution after split-brain.

* Data Storage Layer responsible for indexes and data maintenance

[Architecture follow "shared nothing" peer-to-peer pattern](https://www.aerospike.com/docs/architecture/data-distribution.html). Data are partitioned automatically and shards are distributed evenly across nodes in a cluster. Every node will contain few partitions copies. One node becomes the data master f**or reads and writes for a partition** and **other nodes store replicas**. The replication factor is a configuration parameter.

The client has location awareness for the data (the client knows where each partition is located) so the data can be retrieved in a single hop for the node. Every read and write request is sent to a **data master** for processing. 

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_15.png)

Declared:

- native, multi-threaded, multi-core Flash I/O and an **Aerospike log structured file system** which take advantage of low level SSD read and write patterns. In addition, writes to disk are performed in large blocks to minimize latency, bypassing standard file system like historically tuned to rotational disks.

- built-in are a **Smart Defragmenter** and **Intelligent Evictor**. These processes work together to ensure that there is space in DRAM and that data is never lost and safely written to disk.

- sub-millisecond communication latencies between nodes

# Data consistency model

Declared **strong consistency** due to master replica and synchronous replication until split-brain.

[Also declared that](http://www.aerospike.com/docs/architecture/assets/AerospikeACIDSupport.pdf) "So far, Aerospike has focused on continuous availability and provides the best possible consistency in an AP system by using techniques to avoid partitions (**SIC!**)"

# ACID guarantees

Aerospike [declared ACID guarantees](https://www.aerospike.com/docs/architecture/assets/AerospikeACIDSupport.pdf) and high consistency.

Late [analyze from Jepsen](https://aphyr.com/posts/324-call-me-maybe-aerospike) was published with demonstration of the case when, Aerospike may lost data: "its strongest guarantees are similar to Cassandra or Riak in Last-Write-Wins mode. It may be a safe store for immutable data, but updates to a record can be silently discarded in the event of network disruption. Because Aerospike’s timeouts are so aggressive–on the order of milliseconds–even small network hiccups are sufficient to trigger data loss."

After this publication, Aerospike said that they "[apologize for overstepping the bounds with our marketing pitch and not presenting our consistency claims within the proper technical context](https://discuss.aerospike.com/t/aerospike-loses-data/1250/3)" on their forum.

# Replication schema

For reliability, Aerospike replicates partitions on one or more nodes. One node becomes the data master **for reads and writes** for a partition and other nodes store replicas.

When a node receives a write request, it saves the data AND it forwards the write request to the replica node. Once the replica node confirms that the data has been written successfully AND the node has written the data itself, then confirmation is sent to the client that the write operation was successful. So write to all replicas is synchronous.

The replication factor is a configuration parameter. More replicas mean more reliability, but higher demand on the cluster as write requests must go to all copies of the data.

# Sharding Schema

[Sharding schema is similar to Cassandra with tunable replication factor](http://www.aerospike.com/docs/architecture/data-distribution.html)*.*

Data is distributed evenly across nodes in a cluster using the Aerospike Smart Partitions™ algorithm. Declared "extremely random hash function ensures that partitions are distributed very evenly within a 1-2% error margin".

To determine where a record should go, the record key (of any size) is hashed into a 20-byte fixed length string using RIPEMD160, and the first 12 bits form a partition ID which determines which of the partitions should contain this record. 

Automatic sharding and re-balancing.

# CAP-theorem

[Accordingly to whitepaper](http://pages.aerospike.com/rs/aerospike/images/Acid_Whitepaper.pdf), "In the presence of failures, the cluster can run in one of two modes – AP (available and partition tolerant) or CP (consistent and partition tolerant)", **which is a bit strange** because in the next sentence they states that “In the future, Aerospike will add support for CP mode as an additional deployment option.”. Also in the same paper they said that “Aerospike does not yet support CP mode configuration.”

Accordingly to the same whitepaper, they declared CAP in the same time (**SIC!**), "High consistency on AP mode" **which is ridiculous**. Inside the topic, they explained that “High consistency on AP mode” means “that a replicated Aerospike cluster provides high consistency and high availability during node failures and restarts so long as the cluster does not split into separate partitions” (**SIC!**), so really this is CA. Which is also strange in the light of this discussion “[You Can’t Sacrifice Partition Tolerance](http://codahale.com/you-cant-sacrifice-partition-tolerance/)”

Looks like this is one more item which may lead to "apologize for overstepping the bounds with our marketing pitch" 

# Split-brain and recovery from a split brain

Allows splitbrain state, so availability over consistency, AP from CAP

Two merge policies **may be followed**. Either Aerospike will auto-merge the two data items (default

behavior today) or keep both copies for application to merge later (future) (**SIC!** [Accordingly to an Aerospike forum this feature is not available yet](https://discuss.aerospike.com/t/conflict-resolution-handle-by-application-option-available/1228)).

Auto merge works as follows:

*  TTL (time-to-live) based: The record with the highest TTL wins

* Generation based: The record with the highest generation wins

Application merge works as follows:

*  When two versions of the same data item are available in the cluster, a read of this value will return both versions, allowing the application to resolve the inconsistency.

* The client application – the only entity with knowledge of how to resolve these differences – must then re-write the data in a consistent fashion.

# Rack-aware replication and cross-datacenter replication

[Asynchronous Master-to-Master cross datacenter replication](http://www.aerospike.com/docs/architecture/xdr.html) in [Enterprise Edition](http://www.aerospike.com/products-services/). Also [rack-aware replication supported](http://www.aerospike.com/docs/architecture/rack-aware.html).

Currently [XDR (Cross-DataCenter Replication) is not compatible with Security](http://www.aerospike.com/docs/guide/security.html) (audit and role based access) turned on.

# Fault-tolerance

High-availability and fault tolerance is built on replication and peer-to-peer architecture to survive replica fail. 

The remaining nodes automatically perform data migration to copy the replica partitions to new data masters.

So fault-tolerance depends on number of replica.

# Dynamic scalability

Need to be confirmed.

# Persistence

Declared [Hybrid Storage concept](http://www.aerospike.com/docs/architecture/storage.html), Indexes stored in memory, data on SSD with caches in RAM.

# Query Language, Entry Processor, User Defined Functions and Bulk Writes

Own query language, [AQL](http://www.aerospike.com/docs/tools/aql/), [query-level aggregations are available via UDF](http://www.aerospike.com/docs/guide/aggregation.html)

[Aggregations](http://www.aerospike.com/docs/architecture/secondary-index.html#aggregations) are available via Aggregation Framework (Query records feed to aggregation framework to perform filters, aggregations, etc. On each node, the query result is sent to the UDF sub-system to begin processing the results as a stream of records. The results from each node are then collected by the client application, which may also perform additional operations on the data.)

[UDF on LUA](http://www.aerospike.com/docs/architecture/udf.html) acts as a triggers and executed in the main flow of the transaction. Also may operate on a stream (see aggregations) for MapReduce-like operations.

# Indexes (local and global)

[Primary indexes are in-memory only](http://www.aerospike.com/docs/architecture/primary-index.html), so when aerospike server starts, it walks through data on the storage and creates primary index for all the data partitions, but index may survive fast restart.

[Secondary indexes](http://www.aerospike.com/docs/architecture/secondary-index.html) are local. The query to retrieve results from the secondary index is sent to every cluster node, so distributed query will be used. [Secondary indexes will not survive fast restart](http://www.aerospike.com/docs/guide/query.html).

# Traffic encryption and access control

Aerospike does [not provide encryption of data across the network](http://www.aerospike.com/docs/operations/plan/network/)

Audit and role based access is not available for XDR configuration.

# Business model and commercial support

Community edition is open-source, Enterprise edition is closed-source and available by subscription for data volume.

# Usage experience

We had no experience with this database. 

Colleagues reported easy installation and configuration

In the same time some "overstepping the bounds with our marketing pitch" (see ACID, CAP and split-brain sections) looks very alarming.

# Additional Materials, required to read

[AeroSpike architecture](http://www.aerospike.com/docs/architecture/)

[AeroSpike data model](http://www.aerospike.com/docs/architecture/data-model.html)

[AeroSpike data management](http://www.aerospike.com/docs/architecture/data-management.html)

[AeroSpike FAQ](http://www.aerospike.com/docs/guide/FAQ.html)

[AeroSpike distribution](http://www.aerospike.com/docs/architecture/distribution.html)

[AeroSpike clustering](http://www.aerospike.com/docs/architecture/clustering.html)

[Call me maybe: Aerospike](https://aphyr.com/posts/324-call-me-maybe-aerospike)