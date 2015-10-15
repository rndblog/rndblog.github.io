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
editable: false
---

**MongoDB** is **Open-source** (GNU AGPL v3.0), written in C, CPP and JavaScript. So no freezes due to GC. [Used by](https://www.mongodb.org/community/deployments) Craigslist, eBay, Foursquare, SourceForge, Viacom, The New York Times etc.

This is the one of the few articles in "Notes on NoSQL" sequence  
[Notes on NoSQL : Basics and key differences between relational and NoSQL databases](/nosql/architecture/2015/09/13/notes-on-nosql-basics.html)  
**[Notes on NoSQL : MongoDB, unfulfilled hopes and possible bright future](/nosql/architecture/2015/09/13/notes-on-nosql-mongodb.html)**  
[Notes on NoSQL : Years in production with Apache Cassandra](/nosql/architecture/2015/09/13/notes-on-nosql-cassandra.html)  
[Notes on NoSQL : Apache HBase, database on the top of Hadoop](/nosql/architecture/2015/09/13/notes-on-nosql-hbase.html)  
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
  <h5>This post is a summary, so you have to read all references</h5>
  <p>
    This post is a brief summary and not intended to explain all related terms, but provides references instead. Please read all related materials. If some references missed - please improve the article.
  </p>
</div>

* TOC
{:toc}

**Document-oriented** database with JSON-like ([BSON](https://en.wikipedia.org/wiki/BSON), Binary JSON) documents with dynamic scheme, so **schema is flexible**. Able to operate document part (object field(s)). The maximum BSON document size is 16 megabytes, document can be hierarchical. 

# Architecture

**Distributed architecture** with sharding and replication, every shard contains one Primary and few Secondary replicas. 

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_2.1.png)

# Data consistency model

**Strong data consistency (Linearizability)** is implemented via routing of all Write / Read queries to a Master replica. As an option, Read queries can be issued against secondary copies where data may be **eventually consistent,** if the write operation has not yet been synchronized with the Secondary copy; the consistency choice is made at the query level.

In the same time, Mongo allow to [ask that the primary confirm successful replication of a write](http://docs.mongodb.org/manual/core/write-concern/) by its **disk log**, or by [replication](http://docs.mongodb.org/master/core/replica-set-write-concern/)[ to secondary node(s)](http://docs.mongodb.org/master/core/replica-set-write-concern/). So at the cost of latency we can get **stronger guarantees** about whether or not a write was successful, but still eventually consistent until synchronous replication to all.

Nevertheless, there is a case which is "rare and typically occurs as a result of a network partition with replication lag" when Primary may lost confirmed write, see CAP-theorem section. **So strong consistency guarantee may be violated in case of network partitioning after failover**.

# ACID guarantees

MongoDB declared ACID compliance at the document level

# Replication schema

[Replication](http://docs.mongodb.org/manual/core/replication/) is used to propagate changes from Primary to Secondary replicas. 

Primary replica of a shard (replica set) is [elected (and re-elected)](http://docs.mongodb.org/v3.0/core/replica-set-elections/) by the nodes of this shard. *Priority* configuration parameter and *optime* (timestamp of the last operation that a member applied from the oplog) is considered.

By default, replication is asynchronous: secondary replicas read updates (oplog) from Master asynchronously, so Secondary replicas have **Weak consistency** (**eventually consistent)**. 

Oplog size is limited on primary, so if secondary missed to read oplog update in time (due to network partition or high write throughput) they will be overwritten and lost for Secondary. So at least until 3.0, when Primary is under heavy load, Secondary may miss synchronization. In this case Secondary will be disconnected unrecoverably until manual recovery. Manual recovery will require to stop Primary and resync. Downtime is required to change log size.

# Sharding Schema

[Sharding Cluster](http://docs.mongodb.org/manual/core/sharding/) in MongoDB is optional and [recommended to be used only for high data volume or velocity](http://docs.mongodb.org/manual/core/sharded-cluster-requirements/).

The [shard key is either an indexed field or an indexed compound field](http://docs.mongodb.org/manual/core/sharding-shard-key/) that exists in every document in the collection. 

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_3.png)

MongoDB partitions data in the collection using ranges (chunks) of shard key values. MongoDB distributes the chunks, and their documents, among the shards in the cluster. When a chunk grows beyond the chunk size, MongoDB attempts to split the chunk. 

Sharding and re-sharding is automatic, range-based, hash-based, location-aware.

If all members of a replica set within a shard are unavailable, all data held in that shard is unavailable. If a shard is inaccessible or unavailable, queries will return with an error.

# CAP-theorem

MongoDB is declared as CP-system, this means that in case of network partition database may lose availability, but not consistency.

Nevertheless, [under some corner cases, MongoDB may lose confirmed write](https://aphyr.com/posts/284-call-me-maybe-mongodb) "*if the primary had accepted write operations that the secondaries had not successfully replicated before the primary stepped down. When the primary rejoins the set as a secondary, it reverts, or “rolls back," its write operations to maintain database consistency with the other members.*”

# Split-brain and recovery from a split brain

Split-brain is prevented [by the minority-majority principle](http://docs.mongodb.org/manual/core/replica-set-architectures/). 

Minor set of Secondary replicas without a Primary will NOT elect new Primary if it was lost while partitioning and will demote existing Primary to a secondary, if Primary in minority group. So, minority will be switched to Read-Only.

Major set (N/2+1) of Replicas will continue work with existing Primary or will elect new One.

Because this architecture demotes the original Primary on minority, it will not be affected by  split-brain problem.

# Rack-aware replication and cross-datacenter replication

Accordingly to [MongoDB Multi-Data Center Deployments whitepaper](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Multi_Data_Center.pdf), MongoDB can

- use datacenter-aware sharding to store specific part data in pre-defined data-center

- ensure write operations propagate to specific members of a replica set, deployed locally and in remote data centers

This means that MongoDB have rack-aware and datacenter-aware replication inside a replica set plus datacenter-aware sharding.

There is no cluster-to-cluster replication

# Fault-tolerance

**Fault-tolerance** is built on

* [replica set high availability](http://docs.mongodb.org/manual/core/replica-set-high-availability/) (which is built on [failover](http://docs.mongodb.org/manual/reference/glossary/#term-failover))

* [sharded cluster high-availability](http://docs.mongodb.org/manual/core/sharded-cluster-high-availability/) ([multiple](http://docs.mongodb.org/manual/core/sharded-cluster-components/) [routing](http://docs.mongodb.org/manual/core/sharded-cluster-query-router/) [instances](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos) etc.)

So fault-tolerance [depends on cluster architecture](http://docs.mongodb.org/manual/core/sharded-cluster-architectures-production/) chosen.

[Accordingly to documentation](http://docs.mongodb.org/manual/faq/replica-sets/#how-long-does-replica-set-failover-take), in case of master fail, "replica set will select a new primary **within a minute**". “It may take 10-30 seconds for the members of a replica set to declare a primary inaccessible. This triggers an election. During the election, the cluster is unavailable for writes. The election itself may take another 10-30 seconds.”

# Dynamic scalability

Accordingly to [documentation](http://docs.mongodb.org/v3.0/tutorial/expand-replica-set/) and feedbacks, Secondary replica may be added and removed without downtime.

Accordingly to [documentation](http://docs.mongodb.org/v3.0/tutorial/add-shards-to-shard-cluster/), shards can be added and removed without downtime.

Both operation requires manual operations via mongo shell or [write script](http://docs.mongodb.org/master/tutorial/write-scripts-for-the-mongo-shell/).

# Persistency

Persistence behaviour depends on "[write concern](http://docs.mongodb.org/manual/core/write-concern/)" setting (on a query time!) and this is a trade-off between latency and ability to recover.

[By default](http://docs.mongodb.org/manual/release-notes/drivers-write-concern/#driver-write-concern-change) operation confirmed when [mongod](http://docs.mongodb.org/manual/reference/program/mongod/#bin.mongod) (Primary) confirms that it received the write operation and applied the change to the in-memory view of data

To ensure that MongoDB can recover the data following a shutdown or power interruption, **journaled write concern** (acknowledges the write operation only after committing the data to the [journal](http://docs.mongodb.org/manual/reference/glossary/#term-journal)) or **replica acknowledged write concern (**write operation propagated to additional members of the replica set) may be used.

Starting MongoDB 3.0 compression is supported by the storage engine.

# Query Language, Entry Processor, User Defined Functions and Bulk Writes

Accordingly to [MongoDB Architecture Guide](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Architecture_Guide.pdf) MongoDB support 

* reach query language (KV-queries, range queries, text search) with query optimizations

* aggregation framework for group-by-like queries 

* MapReduce queries on JavaScript

* [bulk writes](http://docs.mongodb.org/v3.0/core/bulk-write-operations/)

# Indexes (local and global)

MongoDB [supports set of indexes](http://docs.mongodb.org/manual/core/index-types/): unique, compound, array, TTL, sparse, text).

Indexes are local, every shard will have its own index (containing just the documents in this shard).  In case of request, which affects few shards, they will be accessed in parallel (every shard reads its own local index shard) and then results merged.

Global indexes are not supported.

# Traffic encryption and access control

[Transport encryption is available via TLS/SSL](http://docs.mongodb.org/manual/tutorial/configure-ssl/) starting 3.0. Certain distributions of MongoDB do not contain support for SSL.

[Encryption at rest](http://docs.mongodb.org/master/core/security-introduction/#encryption-at-rest) is not available.

[Role-Based Access Control](http://docs.mongodb.org/manual/tutorial/manage-users-and-roles/) is available.

MongoDB Enterprise provides support for authentication using SASL and LDAP with [ActiveDirectory](https://docs.mongodb.org/v3.0/tutorial/configure-ldap-sasl-activedirectory/) and [OpenLDAP](https://docs.mongodb.org/v3.0/tutorial/configure-ldap-sasl-openldap/) plus authentication with [Kerberos](https://docs.mongodb.org/v3.0/tutorial/control-access-to-mongodb-with-kerberos-authentication/).

# Business model and commercial support

MongoDB, Inc distributed two versions of MongoDB, commercial version is named Enterprise MongoDB and includes:

* support

* [OpsManager](https://www.mongodb.com/products/ops-manager) product

* advanced security (Kerberos and LDAP etc.)

* commercial license

* platform certification

# Usage experience

By design, MongoDB is a good candidate to migrate from RDBMS. MongoDB was used as a system of record and results are next:

* Traffic encryption with SSL had issues in production until 2.6; after 2.6 tested, results ok, but **not used** in production yet.

* Good monitoring tools, but profiling tools are limited. MongoLab used for management.

* Faced issue with limited oplog size under the load, so at least until 3.0, when cluster was under heavy load, secondary nodes missed synchronization and was disconnected until manual recovery. Manual recovery required to stop primary and resync.

* Incremental update of collection wasn't available until 2.6, after 2.6 fixed for some cases

* [DB wide locks](http://docs.mongodb.org/master/faq/concurrency/#which-administrative-commands-lock-the-database) on index creation/update, create/drop collection, auth, getLastError etc.

* Secondary indexes are slow

* [Safe writes](http://docs.mongodb.org/v3.0/core/write-concern/) are slow, unsafe write ([default](http://docs.mongodb.org/v3.0/core/write-concern/#write-concern-acknowledged), change to the in-memory view of data) may lead to data loss in case of node fail. Also data loss cases [described and shown](https://aphyr.com/posts/284-call-me-maybe-mongodb) in case of cluster split-merge for almost all write concern levels.

* [Stale reads with WriteConcern Majority and ReadPreference Primary](https://aphyr.com/posts/322-call-me-maybe-mongodb-stale-reads) described, when network partitions cause a leader election. [Defect](https://jira.mongodb.org/browse/SERVER-17975) is not fixed yet.

* At least 3 nodes need to be deployed

* Due to different reasons query time may sometimes be increased to 20 seconds instead of average 200 ms.

* Old versions may damage DB files on FS in case of power fail

* Other colleagues reported frequent failover (once a week) with rollbacks

So, by our experience, currently MongoDB may be used as a cache, where data loss is non-critical. In this case it also may achieve good performance results. 

# Additional Materials, required to read

[MongoDB Architecture Guide](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Architecture_Guide.pdf)

[Call me maybe: MongoDB](https://aphyr.com/posts/284-call-me-maybe-mongodb)

[Call me maybe: MongoDB stale reads](https://aphyr.com/posts/322-call-me-maybe-mongodb-stale-reads)

[MongoDB FAQ](https://www.mongodb.com/faq)

[MongoDB Multi-Data Center Deployments](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Multi_Data_Center.pdf)

