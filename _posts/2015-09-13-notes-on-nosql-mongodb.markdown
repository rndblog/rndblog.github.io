---
layout: post
title:  "Notes on NoSQL : MongoDB, unfulfilled hopes and possible bright future"
date:   2015-09-13 15:00:00
update: 2015-09-14 12:55:00
categories: NoSQL Architecture
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Principal Software Engineer and Grid Architect in San Francisco, USA. You can subscribe to my new posts in my <a href="http://sotnikdv.github.io">personal blog</a> or find me in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
editable: true
---

**MongoDB** is **Open-source** (GNU AGPL v3.0) database, written in C, CPP and JavaScript. So no freezes due to GC. [Used by](https://www.mongodb.org/community/deployments) Craigslist, eBay, Foursquare, SourceForge, Viacom, The New York Times etc.

This is the one of the few articles in "Notes on NoSQL" sequence  
[Notes on NoSQL : Basics and key differences between relational and NoSQL databases](/nosql/architecture/2015/09/13/notes-on-nosql-basics.html)  
**[Notes on NoSQL : MongoDB, unfulfilled hopes and possible bright future](/nosql/architecture/2015/09/13/notes-on-nosql-mongodb.html)**  
[Notes on NoSQL : Years in production with Apache Cassandra](/nosql/architecture/2015/09/13/notes-on-nosql-cassandra.html)  
[Notes on NoSQL : Apache HBase, database on the top of Hadoop](/nosql/architecture/2015/09/13/notes-on-nosql-hbase.html)  
[Notes on NoSQL : Redis, pretty simple and jet fast cache](/nosql/architecture/2015/09/13/notes-on-nosql-redis.html)  
[Notes on NoSQL : Aerospike and risqué promises](/nosql/architecture/2015/09/13/notes-on-nosql-aerospike.html)  

<div class="note info">
  <h5>This post is the summary (compilation) from many sources, you have to read all references</h5>
  <p>
    Post is not intended to explain all related terms or re-write all information from origins. So, please consider this post as the "navigation map" and read all references in the text body and in the end of the post.
  </p>
</div>

<div class="note info">
  <h5>A special thanks to <a href="http://griddynamics.com">Grid Dynamics Inc.</a> and colleagues from the author</h5>
  <p>
    This post was initially written as notes in my personal time, in the period when I worked on related topic for Grid Dynamics Inc. <br/>I'd like to thanks to the company and colleagues for the great support and knowledge shared. Personal thanks to <a href="https://ru.linkedin.com/in/dornatsky">Dmitry Ornatsky</a> and <a href="https://www.linkedin.com/in/stryuber">Sergey Tryuber</a>
  </p>
</div>


* TOC
{:toc}

**Document-oriented** database with JSON-like ([BSON](https://en.wikipedia.org/wiki/BSON), Binary JSON) documents with dynamic scheme, so **schema is flexible**. Able to operate document part (object field(s)). The maximum BSON document size is 16 megabytes, document can be hierarchical. 

# Architecture

**Distributed architecture** with **sharding** and **replication**, every shard consists of one Primary and few Secondary replicas. 

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_2.1.png)

# Data consistency model

**Strong data consistency (Linearizability)** is implemented via routing of all Write / Read queries to a Master replica. As an option, Read queries can be issued against secondary replicas, where data may be **eventually consistent** (if the write operation has not yet been synchronized with the Secondary copy); the consistency choice is made at the query level.

In the same time, Mongo allow to [ask that the primary confirm successful replication of a write](http://docs.mongodb.org/manual/core/write-concern/) by its **disk log**, or by [replication](http://docs.mongodb.org/master/core/replica-set-write-concern/)[ to secondary node(s)](http://docs.mongodb.org/master/core/replica-set-write-concern/). So, at the cost of latency we can get **stronger guarantees** about whether or not a write was successful, but such model still eventually consistent until synchronous replication to all used.

Nevertheless, there is a case which is "rare and typically occurs as a result of a network partition with replication lag" when Primary may lost confirmed write, see CAP-theorem section. **So, strong consistency guarantee may be violated in case of network partitioning after failover**.

# ACID guarantees

MongoDB declared ACID compliance at the document level

# Replication schema

[Replication](http://docs.mongodb.org/manual/core/replication/) is used to propagate changes from Primary to Secondary replicas. 

Primary replica of a shard (replica set) is [elected (and re-elected)](http://docs.mongodb.org/v3.0/core/replica-set-elections/) by the nodes of this shard. *Priority* configuration parameter and *optime* (timestamp of the last operation that a member applied from the oplog) is used while election.

By default, replication is asynchronous: secondary replicas read updates (oplog) from Master asynchronously, so Secondary replicas have **weak (eventual) consistency**. 

*Oplog* size is limited on primary replica node, so, if secondary replica node missed to read oplog update in time (due to network partition or high write throughput), it will be overwritten and lost for Secondary. At least until version 3.0, when Primary is under heavy load, Secondary may miss synchronization. In this case Secondary will be disconnected and need to be recovered manually. Such manual recovery will require to stop Primary and resync. Downtime is also required to change log size.

# Sharding Schema

[Sharding Cluster](http://docs.mongodb.org/manual/core/sharding/) in MongoDB is optional and [recommended to be used only for high data volume or velocity](http://docs.mongodb.org/manual/core/sharded-cluster-requirements/).

The [shard key is either an indexed field or an indexed compound field](http://docs.mongodb.org/manual/core/sharding-shard-key/) that exists in every document in a collection. 

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_3.png)

MongoDB splits data in a collection to partitions using ranges (chunks) of shard key values. MongoDB distributes chunks, and their documents, among a shards in a cluster. When a chunk grows beyond a chunk size, MongoDB will split chunk. 

Sharding and re-sharding can be automatic, range-based, hash-based, location-aware.

If all members of a replica set within a shard are unavailable, all data held in that shard is unavailable. If a shard is inaccessible or unavailable, queries will return with an error.

# CAP-theorem

MongoDB is declared as CP-system, in case of network partition database may lose availability, but not consistency.

Nevertheless, [under some corner cases, MongoDB may lose confirmed write](https://aphyr.com/posts/284-call-me-maybe-mongodb) "*if the primary had accepted write operations that the secondaries had not successfully replicated before the primary stepped down. When the primary rejoins the set as a secondary, it reverts, or “rolls back," its write operations to maintain database consistency with the other members.*”

# Split-brain and recovery from a split brain

Split-brain is prevented [by the minority-majority principle](http://docs.mongodb.org/manual/core/replica-set-architectures/). 

Minor set of Secondary replicas without a Primary will NOT elect new Primary if it was lost while network partitioning and will demote existing Primary to a secondary, if Primary is in such minor set. So, minority group will be switched to Read-Only.

Major set (N/2+1) of Replicas will continue work with existing Primary or will elect new one, if it was lost.

So MongoDB will not be affected by the split-brain problem, because of it demotes original Primary in minority.

# Rack-aware replication and cross-datacenter replication

Accordingly to [MongoDB Multi-Data Center Deployments whitepaper](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Multi_Data_Center.pdf), MongoDB can

- use datacenter-aware sharding to store specific part data in pre-defined data-center

- ensure write operations propagate to specific members of a replica set, deployed locally and in remote data centers

MongoDB have rack-aware and datacenter-aware replication inside a replica set plus datacenter-aware sharding.

There is no cluster-to-cluster replication

# Fault-tolerance

**Fault-tolerance** is based on

* [replica set high availability](http://docs.mongodb.org/manual/core/replica-set-high-availability/) (which is based on [failover](http://docs.mongodb.org/manual/reference/glossary/#term-failover))

* [sharded cluster high-availability](http://docs.mongodb.org/manual/core/sharded-cluster-high-availability/) ([multiple](http://docs.mongodb.org/manual/core/sharded-cluster-components/) [routing](http://docs.mongodb.org/manual/core/sharded-cluster-query-router/) [instances](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos) etc.)

So fault-tolerance [depends on the cluster deployment schema](http://docs.mongodb.org/manual/core/sharded-cluster-architectures-production/) chosen.

[Accordingly to documentation](http://docs.mongodb.org/manual/faq/replica-sets/#how-long-does-replica-set-failover-take), in case of master fail, "replica set will select a new primary **within a minute**". “It may take 10-30 seconds for the members of a replica set to declare a primary inaccessible. This triggers an election. During the election, the cluster is unavailable for writes. The election itself may take another 10-30 seconds.”

# Dynamic scalability

Accordingly to [documentation](http://docs.mongodb.org/v3.0/tutorial/expand-replica-set/) and feedbacks, Secondary replica may be added and removed without downtime.

Accordingly to [documentation](http://docs.mongodb.org/v3.0/tutorial/add-shards-to-shard-cluster/), shards can be added and removed without downtime.

Both operation requires manual operations via mongo shell or [write script](http://docs.mongodb.org/master/tutorial/write-scripts-for-the-mongo-shell/).

# Persistency

Persistency behaviour depends on "[write concern](http://docs.mongodb.org/manual/core/write-concern/)" setting (on a query time!) and this is a trade-off between latency and ability to recover data in case of node/network fail.

[By default](http://docs.mongodb.org/manual/release-notes/drivers-write-concern/#driver-write-concern-change) operation confirmed when [mongod](http://docs.mongodb.org/manual/reference/program/mongod/#bin.mongod) (Primary) confirms that it received the write operation and applied the change to the in-memory view of data.

To ensure that MongoDB can recover the data following a shutdown or power interruption, **journaled write concern** (acknowledges the write operation only after committing the data to the [journal](http://docs.mongodb.org/manual/reference/glossary/#term-journal)) or **replica acknowledged write concern (**write operation propagated to additional members of the replica set) may be used.

Starting MongoDB 3.0 compression is supported by the storage engine.

# Query Language, Entry Processor, User Defined Functions and Bulk Writes

Accordingly to [MongoDB Architecture Guide](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Architecture_Guide.pdf) MongoDB supports: 

* reach query language (KV-queries, range queries, text search) with query optimizations

* aggregation framework for group-by-like queries 

* MapReduce queries on JavaScript

* [bulk writes](http://docs.mongodb.org/v3.0/core/bulk-write-operations/)

# Indexes (local and global)

MongoDB [supports set of indexes](http://docs.mongodb.org/manual/core/index-types/): unique, compound, array, TTL, sparse, text.

Indexes are local, every shard have own index (contains just documents from this shard).  

In case of request, which affects few shards, such shards will be requested in parallel (and every shard will use own local index) and then results will be merged.

Global indexes are not supported.

# Traffic encryption and access control

[Transport encryption is available via TLS/SSL](http://docs.mongodb.org/manual/tutorial/configure-ssl/) starting 3.0. Certain distributions of MongoDB do not contain support for SSL.

[Encryption at rest](http://docs.mongodb.org/master/core/security-introduction/#encryption-at-rest) is not available.

[Role-Based Access Control](http://docs.mongodb.org/manual/tutorial/manage-users-and-roles/) is available.

MongoDB Enterprise provides support for authentication using SASL and LDAP with [ActiveDirectory](https://docs.mongodb.org/v3.0/tutorial/configure-ldap-sasl-activedirectory/) and [OpenLDAP](https://docs.mongodb.org/v3.0/tutorial/configure-ldap-sasl-openldap/) plus authentication with [Kerberos](https://docs.mongodb.org/v3.0/tutorial/control-access-to-mongodb-with-kerberos-authentication/).

# Business model and commercial support

MongoDB, Inc. distributed two versions of MongoDB, commercial version is named Enterprise MongoDB and includes:

* support

* [OpsManager](https://www.mongodb.com/products/ops-manager) product

* advanced security (Kerberos and LDAP etc.)

* commercial license

* platform certification

# Use experience

MongoDB may be a good candidate to migrate from RDBMS, if you need to store documents. MongoDB was used as a system of record and results are next:

* Traffic encryption with SSL had issues in production until 2.6; after 2.6 tested, results ok, but **not used** in production yet;

* Good monitoring tools, but profiling tools are limited. MongoLab used for management;

* Faced issue with limited oplog size (described in Replication schema section) under the load;

* Incremental update of collection wasn't available until 2.6, after 2.6 fixed for some cases;

* [DB wide locks](http://docs.mongodb.org/master/faq/concurrency/#which-administrative-commands-lock-the-database) on index creation/update, create/drop collection, auth, getLastError etc.;

* Secondary indexes are slow;

* [Safe writes](http://docs.mongodb.org/v3.0/core/write-concern/) are slow, unsafe writes ([default](http://docs.mongodb.org/v3.0/core/write-concern/#write-concern-acknowledged)) may lead to data loss if master node fail. Also possible data loss case [described and shown](https://aphyr.com/posts/284-call-me-maybe-mongodb) for cluster split-merge on almost all write concern levels;

* [Stale reads with WriteConcern Majority and ReadPreference Primary](https://aphyr.com/posts/322-call-me-maybe-mongodb-stale-reads) described, when network partitions caused election. [Defect](https://jira.mongodb.org/browse/SERVER-17975) is not fixed yet;

* At least 3 nodes need to be deployed;

* Query time spikes up to 20 seconds (instead of average 200 ms.);

* Old versions may damage DB files on FS in case of power fail;

* Colleagues reported frequent failover (once a week) with rollbacks

So, by our experience, currently MongoDB may be used as a cache, where data loss is non-critical. In this case it also may achieve good performance results. 

# Additional Materials, recommended to read

[MongoDB Architecture Guide](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Architecture_Guide.pdf)

[Call me maybe: MongoDB](https://aphyr.com/posts/284-call-me-maybe-mongodb)

[Call me maybe: MongoDB stale reads](https://aphyr.com/posts/322-call-me-maybe-mongodb-stale-reads)

[MongoDB FAQ](https://www.mongodb.com/faq)

[MongoDB Multi-Data Center Deployments](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Multi_Data_Center.pdf)

