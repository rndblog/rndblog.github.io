---
layout: post
title:  "Notes on NoSQL and NoSQL DBs"
date:   2015-09-13 04:28:00
categories: NoSQL Architecture
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Grid Architect at GridDynamics in San Francisco, USA. You can find me also in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
editable: false
---

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

<div class="note warning">
  <h5>Temporary address</h5>
  <p>
    This post will be moved soon from this address to a new location. Please do not distribute link to this post and do not add link to it to any resources.
  </p>
</div>


* TOC
{:toc}

# Difference between Relational and NoSQL databases

*In partitioned databases, trading some consistency for availability can lead to dramatic improvements in scalability.*

*Dan Pritchett, eBay*

In general, relational and NoSQL databases it’s a different solutions in a trade-off between ACID guarantees, durability, integrity (transactional and referential) and strong consistency on one side and low latency, horizontal scalability, throughput and high availability on the other side.

Different architectural solutions and tunable settings of Relational (RDBMS) and NoSQL databases may provide various balance between these corner cases, but this is always kind of trade-off.

In general, RDBMS and NoSQL databases have different areas of applicability, where their strengths are fully used and impact from weakness is minimal: [System of Record](https://en.wikipedia.org/wiki/System_of_record) and [Big Data](https://en.wikipedia.org/wiki/Big_data) Pros and cons of RDBMS and NoSQL systems it’s a direct consequences of their area of applicability.

While requirements of a [System of Record](https://en.wikipedia.org/wiki/System_of_record) is widely known, let’s highlight key challenges of [Big Data](https://en.wikipedia.org/wiki/Big_data) area:

* **Velocity** - query/updates are exceptionally fast or large

* **Volume** - store a massive amount of data at rest

* **Variety** - The structure of records varies dramatically

So, there is no silver bullet or universal solution for every case. Every tool and configuration of a tool need to be chosen for particular case accordingly to project’s requirements. So question "What’s better, Relational or NoSQL database?" is meaningless until will be followed by “... for this particular case”.

## Distributed and Centralized architecture

The key architectural difference between RDBMS and NoSQL databases is a **distributed** (NoSQL) and **centralized** approach (RDBMS). Most other characteristics (positive and negative) are result of the solution used.

Distributed storage (and Sharding) is a key architectural solution to 2 of 3 Big Data challenges: **Velocity** and **Volume**.

This is **not true** that CAP theorem prohibits to create a distributed database with ACID transaction capability, but **strong consistency** and **ACID guarantees** with **transactions** support will seriously affect performance of a distributed database and, literally, eliminate almost all benefits from distributed architecture. 

## Horizontal scalability

**Distributed** applications are more **scalable**, than **centralized**. So NoSQL database usually are **horizontally scalable** at **near-linear scale** for data volume and throughput (with data partitioning, sharding and replication used).

This make NoSQL databases good to deal with **high data volume** and **high data velocity**.

## Transactions

**Transactions** are much more expensive, complex in a distributed environment and may affect performance, so this is one more trade-off with performance. This is the reason why transactions are, in general, not supported in NoSQL (*Some NoSQL implementations may offer similar functionality. For example, Oracle's NoSQL offers transactional control over data written to one node and let choose consistency across multiple nodes. For perfect consistency it will wait for each write to reach all nodes*).

Some NoSQL databases (for example Intersystems Cache) may provide full transaction support but with significant impact to performance, as was explained above.

Nevertheless, NoSQL database provide atomicity on for a single record change.

## Relational and non-relational data model

RDBMS are based on relational model of data with pre-defined schema. This  is a key for referential data consistency and normalization in RDBMS. In RDBMS data are normalized and stored in a set of tables with relations and referential integrity constraints.

NoSQL is based on record-centric approach (document oriented, graph, key-value, wide column etc.) which is a good for data partitioning, scalability and fast read-write. As a result, in NoSQL data stored in de-normalized form in a few independent tables. So there is no links to other keys, no relations between records, no automatic reference constraint checks.

In NoSQL data model usually have no pre-defined schema, schema is **flexible**. Data types are supported. On one hand this is a easy way to deal with 3rd challenge of Big Data - **Data Variety** and schema change, on the other hand pre-defined schema may guarantee more data consistency.

## Indexes support

As well as transactions, indexes implementation and maintenance is more complex in distributed databases and on data models with flexible schema. This means that NoSQL indexes support usually less advanced than in RDBMS due to trade-off with performance.

In most NoSQL databases indexes are local (each node store indexes for local data) and distributed over the cluster, so cross-partition search request will be sent to multiple (all) nodes in parallel. Some NoSQL databases may provide global indexes (which cover all data partitions) with different limitations.

## How to query and manipulate data

RDBMS and relational data model is good for heavy queries, which operate multiple data tables with complex conditions, aggregations and different transaction isolation levels.

NoSQL is good for series of lightweight record read-write queries by a key and usually doesn’t provide aggregations and filtering. Also atomicity is supported on record level.

Since most NoSQL databases lack ability for joins in queries, the database schema generally needs to be designed differently. [There are three main techniques for handling relational data in a NoSQL database](https://en.wikipedia.org/wiki/NoSQL#Handling_relational_data):

* **Multiple queries**. Instead of retrieving all the data with one query, it's common to do several queries to get the desired data. NoSQL queries are often faster than traditional SQL queries so the cost of having to do additional queries may be acceptable. 

* **Caching/replication/non-normalized data**. Instead of only storing foreign keys, it's common to store actual foreign values along with the model's data. For example, each blog comment might include the username in addition to a user id, thus providing easy access to the username without requiring another lookup. When a username changes however, this will now need to be changed in many places in the database. Thus this approach works better when reads are much more common than writes.

* **Nesting data**. With document databases like MongoDB it's common to put more data in a smaller number of collections. For example in a blogging application, one might choose to store comments within the blog post document so that with a single retrieval one gets all the comments. Thus in this approach a single document contains all the data you need for a specific task.

To simplify data access NoSQL databases may offer **Query Language**, **Entry Processors** and **Aggregation Frameworks**.

### Query Language in NoSQL

Many NoSQL databases provide different **Query Languages** which allow to query/update multiple records with filtering. Such **Query Language** usually provides significantly limited subset (no joining or aggregation functions) of SQL functionality due to trade-off with performance in distributed systems. Instead of this, some NoSQL databases may offer **Entry Processors** and **Aggregation Framework**, see below.

Some NoSQL databases may provide SQL92, but, as was discussed, this will significantly decrease performance.

### Entry Processors in NoSQL

**Entry Processor** is a custom code unit which is executed inside a database for bulk data processing on the node, which store a data. This feature may be provided by some NoSQL databases for analytics and data mining.

Also **Entry Processors** may process data streams, returned by query on Query Language or may be triggered on write event.

### Aggregation Framework in NoSQL

Some NoSQL databases may offer **Aggregation Frameworks** which extends abilities of Query Language for complex analytics tasks on data stored. See [MongoDB Aggregation Examples](http://docs.mongodb.org/manual/applications/aggregation/). 

## Sharding

**Sharding** is a result of distributed architecture of NoSQL databases and data model used, and key to scalability. A **database shard** is a [horizontal partition](https://en.wikipedia.org/wiki/Partition_(database)) of data. **Horizontal partitioning** is a database design principle whereby **rows** of a database table are held separately, rather than being split into columns (which is what normalization and vertical partitioning do, to differing extents). Each **partition** forms part of a **shard**, each **shard** is held on a **separate database server instance**, to spread load.

By [shared nothing architecture](https://en.wikipedia.org/wiki/Shared_nothing_architecture) - once sharded, each shard can live in a totally separate logical schema instance / physical database server / data center / continent. This makes replication across multiple servers easy. 

As was discussed, on the other side this architecture approach make hard to maintain indexes and queries, which require data from different shards.

Different NoSQL databases provide different sharding algos, configurable and automatic.

Some RDBMS may offer sharding, but in NoSQL this is key architectural property.

## Replication

To achieve **High Availability** and **Disaster Recovery** many databases (Relational and NoSQL) use **Replication**.

In NoSQL databases shards usually **replicated** between **cluster nodes** and this is one of the most important trade-offs (usually configurable) between consistency and performance in NoSQL world. Two parameters are most important: replicas count and how many replicas should be updated before write will be confirmed to client.

Different NoSQL databases may use master-slave and peer-to-peer replication inside a cluster.

Some databases offer **rack-aware replication**, which tracks that replicas will in in different availability groups (racks or availability zones) 

As for **cluster-to-cluster** (**cross-datacenter replication**, **XDR**), this functionality may be present in RDBMS and NoSQL databases.

## CAP theorem as a result of distributed storage

The main problem of **distributed storage** such as NoSQL databases is a consistency and availability balance while network partitioning. 

[CAP theorem](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed) states that it is impossible for a distributed computer system to simultaneously provide all three of the following guarantees:

* Consistency, equivalent to having a single up-to-date copy of the data

* Availability of that data (for updates)

* Partition tolerance, ability of the system to operate when partitioned (due to network issues)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_0.png)

[In other word’s CAP theorem only says](https://aphyr.com/posts/313-strong-consistency-models) that **we cannot build totally available linearizable systems**. The problem is that we have other proofs which tell us that we cannot build totally available systems with sequential, serializable, repeatable read, snapshot isolation, or cursor stability – or any models stronger than those. 

In this map from [Peter Bailis' HAT not CAP paper](http://db.cs.berkeley.edu/papers/vldb14-hats.pdf), models shaded in red cannot be fully available.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_1.png)

Highly available models (yellow):

* Read Uncommitted (RU)

*  Read Committed (RC)

*  Monotonic Atomic View (MAV)

* Item Cut Isolation (I-CI) 

* Predicate Cut Isolation (PCI)

* Writes Follow Reads (WFR)

*  Monotonic Reads (MR)

*  Monotonic Writes (MW)

Sticky available models (blue):

* Read Your Writes (RYW)

* PRAM

* Causal

Unavailable models (red)

* Cursor Stability (CS)

* Snapshot Isolation (SI)

* Repeatable Read (RR)

* One-Copy Serializability (1SR)

* Strong 1SR

* Recency

* Linearizability

* Safe

* Regular

Most of the NoSQL databases may be configured to have different balance in this trade-off and even in different conditions. For example Cassandra (which is marked as AP) can be also CP-system.

Non-distributed (non-partitionable) storages are not affected by this trade-off.

See [Paxos protocols](https://en.wikipedia.org/wiki/Paxos_(computer_science))

### Split-brain for AP-systems

If system designed to be AP-compatible (available for data change and operate while partitioned), then it will be inconsistent and may be divided onto few partitions. Such state is named [split-brain](https://en.wikipedia.org/wiki/Split-brain_(computing)). [The challenging case for designers of AP-systems is to mitigate a par­tition’s effects on consistency and availability](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed). The key idea is to manage partitions very explicitly, including not only detection, but also a specific [recovery (merge) process](http://cdn.infoq.com/statics_s1_20150819-0313/resource/articles/cap-twelve-years-later-how-the-rules-have-changed/en/resources/fig1%20small.jpg) and a plan for all of the invariants that might be violated during a partition.

## ACID and BASE

[ACID (Atomicity, Consistency, Isolation, Durability)](https://en.wikipedia.org/wiki/ACID) is a set of properties that guarantee that database transactions are processed reliably. In the context of databases, a single logical operation on the data is called a transaction.

Here is a summary fo how the ACID properties are interpreted by a relational DBMS:

* **Atomicity** requires that each transaction is executed in its entirety, or fail without any change being applied.

* **Consistency** requires that the database only passes from a valid state to the next one, without intermediate points.

* **Isolation** requires that if transactions are executed concurrently, the result is equivalent to their serial execution. A transaction cannot see the partial result of the application of another one.

* **Durability** means that the the result of a committed transaction is permanent, even if the database crashes immediately or in the event of a power loss.

So under CAP-theorem restriction, distributed system can provide BASE guarantees:

* **Basically Available**: system does guarantee the availability of the data as regards CAP-theorem; there will be a response to any request. But, that response could still be ‘failure’ to obtain the requested data or the data may be in an inconsistent or changing state.

* **Soft state**: state of the system could change over time, so even during times without input there may be changes going on due to ‘eventual consistency’

* **Eventual consistency**: The system will eventually become consistent once it stops receiving input. The data will propagate to everywhere it should sooner or later, but the system will continue to receive input and is not checking the consistency of every transaction before it moves onto the next one. 

Main difference is that RDBMS can provide ACID guarantees. 

NoSQL, as a distributed system, can provide BASE and ACID under some configurations and restrictions.

## Continuous data availability

"Continuous data availability" is a next step after “High Data availability”, where unplanned downtime, although not desired, is still expected. 

In NoSQL databases this become possible because of distributed architecture, share-nothing approach and replication (**rack-aware** and **cross-datacenter**)

## Persistence to disk

There is different approaches when to confirm data write, usually this is tunable and this is one more trade-off between reliability and performance. 

Most RDBMS confirm write when data a persisted to file storage (data files, commit logs etc.). 

Most NoSQL databases confirm write when changes are applied in memory plus (for some systems and settings) are distributed over few nodes. Writes to disk usually batched and asynchronous for better performance.

It’s a highly discussible question is synchronous replication over the cluster(s) vs disk persistency.

# Common knowledge about NoSQL

## NoSQL Database Types

### Document-oriented databases

Pair each key with a complex data structure known as a document. Documents can contain many different key-value pairs, or key-array pairs, or even nested documents. The notion of a schema is dynamic: each document can contain different fields. 

Document databases provide the ability to query on any field within a document. 

Some products provide indexing options to optimize queries, some of these products provide the ability to analyze data in place, without it needing to be replicated to dedicated analytics or search engines. 

Examples: **MongoDB** and **CouchDB**

### **Graph stores** 

Use graph structures with nodes, edges and properties to represent data. In essence, data is modeled as a network of relationships between specific elements. Mostly used to store information about networks, such as social connections. 

Tend to provide rich query models where simple and complex relationships can be interrogated to make direct and indirect inferences about the data in the system. Relationship-type analysis tends to be very efficient in these systems, whereas other types of analysis may be less optimal. As a result, graph databases are rarely used for more general purpose operational applications.

Examples: **Neo4J, Giraph** and **HyperGraphDB**.

### **Key-value stores** 

Most basic type of non-relational database. Every item in the database is stored as an attribute name, or key, together with its value. This model can be useful for representing polymorphic and unstructured data, as the database does not enforce a set schema across key-value pairs. 

Provide the ability to retrieve and update data based only on a single or a limited range of primary keys. 

For querying on other values, users are encouraged to build and maintain their own indexes. In this case, to perform an update in these systems, multiple round trips may be necessary: first find the record, then update it, then update the index. Some products provide limited support for secondary indexes. 

Update may be implemented as a complete rewrite of the entire record irrespective of whether a single attribute has changed, or the entire record.

Examples: **Riak**, **Voldemort** and **Redis**.

### Wide-column stores (column family stores)

Use a sparse, distributed multi-dimensional sorted map to store data. Each record can vary in the number of columns that are stored. Columns can be grouped together for access in column families, or columns can be spread across multiple column families. Data is retrieved by primary key per column family. **Cassandra** and **HBase** are optimized for queries over large datasets, and store columns of data together, instead of rows.

As for querying and update, the same concept as for key-value stores

Examples: **Cassandra** and **HBase**

## Consistency in NoSQL

Most non-relational systems typically maintain multiple copies of the data for **availability** and **scalability** purposes. These databases can impose different guarantees on the **consistency** of the data across copies. 

Non-relational databases tend to be categorized as either **consistent** or **eventually consistent**. 

With a consistent system, writes by the application are immediately visible in subsequent queries from all copies. With an eventually consistent system writes are not immediately visible from all copies, but system may become consistent later (there is a period of time in which all copies of the data are not synchronized). 

Most NoSQL solutions provide **tunable trade-off** between **consistency** on one side and **availability**, **scalability** and **latency** on the other side.

Many systems may hide from the client internal eventual consistency by the architectural solutions and configuration. Nevertheless, keep in mind above-mentioned trade-off.

For example:

- Cassandra may use quorum for read-write, which affects latency, but allow to keep consistency until tunable threshold of lost nodes.

- In MongoDB, by default, data is consistent because all writes and reads access the primary copy of the data. As an option, read queries can be issued against secondary copies where data maybe eventually consistent if the write operation has not yet been synchronized with the

secondary copy; the consistency choice is made at the query level. In case of primary **fail**, data may become inconsistent.

Need to distinguish **view consistency** and **replica consistency**. **Replicas** in NoSQL solutions is a mostly **eventually consistent**.

## Merge in eventually consistent systems

**Eventually consistent systems** must be able to accommodate conflicting updates in individual records, **if writes can be applied to any copy of the data**, it is possible and not uncommon for writes to conflict with one another. 

Some systems like **Riak** use vector clocks to determine **the ordering** of events and to ensure that the most recent operation wins in the case of a conflict. 

Other systems like **CouchDB** **retain all conflicting values** and push the responsibility to resolving conflict back to the user. 

Another approach, followed by **Cassandra**, is simply to assume the **latest value is the correct one**. 

For these reasons, inserts tend to perform well in eventually consistent systems, but updates and deletes can involve trade-offs.

## APIs of NoSQL databases

Most of the NoSQL databases provides their own API and bindings to different languages.

Also some of them may provide **REST**, [Apache Thrift](https://thrift.apache.org/) and **Memcached** protocol.

**JPA** is available through **ORM** frameworks like **Hibernate** or **EclipseLink.**

## Strengths of NoSQL

* Near-Linear  horizontal scalability for data volume and queries volume

* Compromise between consistency and latency, configurable in many solutions

* Easy High Availability

* Dynamic Schema (some solutions) or at least more flexible schema

* Auto-sharding

* Remote code execution on the node, which contain data (entry processors, map-reduce) (some solutions)

* Automatic replication (master slave, peer-to-peer, XDC)

* Aggregation Frameworks

* Flexible schema

* Continuous data availability

## Weakness of NoSQL

* Basic support of transactions

* Basic support of indexes

* Limited support or absence of Query Language

* Non-relational model with limited integrity check abilities

* CAP-theorem limitations

* Weak consistency for some settings and databases

* Asynchronous persistency to disk vs performance

## Key characteristic of NoSQL DB

Here is the list of NoSQL database characteristics which need to be considered while choosing NoSQL database:

* Architecture

* Database Type

* Is JSON type supported

* CAP-type

* Consistency type

* ACID guarantees

* Sharding schema

* Replication schema

* Is rack-aware replication available

* Is XDR replication available and type (master-master or master-slave)

* Dynamic scalability

* Persistency to disk schema

* Fault-tolerance

* Is Query Language available

* Is Entry Processor and User Defined Functions supported

* Is Aggregation functionality available

* Indexes support (local and global)

* Change merge policy plus split-brain recovery merge policy for AP-types

* Traffic encryption and access control

* Caching schema, if written on Java, then are caches off-heap or on-heap

* License type and business model

* Is commercial support available

# Popular NoSQL solutions overview

## MongoDB

**Open-source** (GNU AGPL v3.0), written in C, CPP and JavaScript. So no freezes due to GC.

[Used by](https://www.mongodb.org/community/deployments) Craigslist, eBay, Foursquare, SourceForge, Viacom, The New York Times etc.

**Document-oriented** database with JSON-like ([BSON](https://en.wikipedia.org/wiki/BSON), Binary JSON) documents with dynamic scheme, so **schema is flexible**. Able to operate document part (object field(s)). The maximum BSON document size is 16 megabytes, document can be hierarchical. 

### Architecture

**Distributed architecture** with sharding and replication, every shard contains one Primary and few Secondary replicas. 

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_2.png)

### Data consistency model

**Strong data consistency (Linearizability)** is implemented via routing of all Write / Read queries to a Master replica. As an option, Read queries can be issued against secondary copies where data may be **eventually consistent,** if the write operation has not yet been synchronized with the Secondary copy; the consistency choice is made at the query level.

In the same time, Mongo allow to [ask that the primary confirm successful replication of a write](http://docs.mongodb.org/manual/core/write-concern/) by its **disk log**, or by [replication](http://docs.mongodb.org/master/core/replica-set-write-concern/)[ to secondary node(s)](http://docs.mongodb.org/master/core/replica-set-write-concern/). So at the cost of latency we can get **stronger guarantees** about whether or not a write was successful, but still eventually consistent until synchronous replication to all.

Nevertheless, there is a case which is "rare and typically occurs as a result of a network partition with replication lag" when Primary may lost confirmed write, see CAP-theorem section. **So strong consistency guarantee may be violated in case of network partitioning after failover**.

### ACID guarantees

MongoDB declared ACID compliance at the document level

### Replication schema

[Replication](http://docs.mongodb.org/manual/core/replication/) is used to propagate changes from Primary to Secondary replicas. 

Primary replica of a shard (replica set) is [elected (and re-elected)](http://docs.mongodb.org/v3.0/core/replica-set-elections/) by the nodes of this shard. *Priority* configuration parameter and *optime* (timestamp of the last operation that a member applied from the oplog) is considered.

By default, replication is asynchronous: secondary replicas read updates (oplog) from Master asynchronously, so Secondary replicas have **Weak consistency** (**eventually consistent)**. 

Oplog size is limited on primary, so if secondary missed to read oplog update in time (due to network partition or high write throughput) they will be overwritten and lost for Secondary. So at least until 3.0, when Primary is under heavy load, Secondary may miss synchronization. In this case Secondary will be disconnected unrecoverably until manual recovery. Manual recovery will require to stop Primary and resync. Downtime is required to change log size.

### Sharding Schema

[Sharding Cluster](http://docs.mongodb.org/manual/core/sharding/) in MongoDB is optional and [recommended to be used only for high data volume or velocity](http://docs.mongodb.org/manual/core/sharded-cluster-requirements/).

The [shard key is either an indexed field or an indexed compound field](http://docs.mongodb.org/manual/core/sharding-shard-key/) that exists in every document in the collection. 

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_3.png)

MongoDB partitions data in the collection using ranges (chunks) of shard key values. MongoDB distributes the chunks, and their documents, among the shards in the cluster. When a chunk grows beyond the chunk size, MongoDB attempts to split the chunk. 

Sharding and re-sharding is automatic, range-based, hash-based, location-aware.

If all members of a replica set within a shard are unavailable, all data held in that shard is unavailable. If a shard is inaccessible or unavailable, queries will return with an error.

### CAP-theorem

MongoDB is declared as CP-system, this means that in case of network partition database may lose availability, but not consistency.

Nevertheless, [under some corner cases, MongoDB may lose confirmed write](https://aphyr.com/posts/284-call-me-maybe-mongodb) "*if the primary had accepted write operations that the secondaries had not successfully replicated before the primary stepped down. When the primary rejoins the set as a secondary, it reverts, or “rolls back," its write operations to maintain database consistency with the other members.*”

### Split-brain and recovery from a split brain

Split-brain is prevented [by the minority-majority principle](http://docs.mongodb.org/manual/core/replica-set-architectures/). 

Minor set of Secondary replicas without a Primary will NOT elect new Primary if it was lost while partitioning and will demote existing Primary to a secondary, if Primary in minority group. So, minority will be switched to Read-Only.

Major set (N/2+1) of Replicas will continue work with existing Primary or will elect new One.

Because this architecture demotes the original Primary on minority, it will not be affected by  split-brain problem.

### Rack-aware replication and cross-datacenter replication

Accordingly to [MongoDB Multi-Data Center Deployments whitepaper](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Multi_Data_Center.pdf), MongoDB can

- use datacenter-aware sharding to store specific part data in pre-defined data-center

- ensure write operations propagate to specific members of a replica set, deployed locally and in remote data centers

This means that MongoDB have rack-aware and datacenter-aware replication inside a replica set plus datacenter-aware sharding.

There is no cluster-to-cluster replication

### Fault-tolerance

**Fault-tolerance** is built on

* [replica set high availability](http://docs.mongodb.org/manual/core/replica-set-high-availability/) (which is built on [failover](http://docs.mongodb.org/manual/reference/glossary/#term-failover))

* [sharded cluster high-availability](http://docs.mongodb.org/manual/core/sharded-cluster-high-availability/) ([multiple](http://docs.mongodb.org/manual/core/sharded-cluster-components/) [routing](http://docs.mongodb.org/manual/core/sharded-cluster-query-router/) [instances](http://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos) etc.)

So fault-tolerance [depends on cluster architecture](http://docs.mongodb.org/manual/core/sharded-cluster-architectures-production/) chosen.

[Accordingly to documentation](http://docs.mongodb.org/manual/faq/replica-sets/#how-long-does-replica-set-failover-take), in case of master fail, "replica set will select a new primary **within a minute**". “It may take 10-30 seconds for the members of a replica set to declare a primary inaccessible. This triggers an election. During the election, the cluster is unavailable for writes. The election itself may take another 10-30 seconds.”

### Dynamic scalability

Accordingly to [documentation](http://docs.mongodb.org/v3.0/tutorial/expand-replica-set/) and feedbacks, Secondary replica may be added and removed without downtime.

Accordingly to [documentation](http://docs.mongodb.org/v3.0/tutorial/add-shards-to-shard-cluster/), shards can be added and removed without downtime.

Both operation requires manual operations via mongo shell or [write script](http://docs.mongodb.org/master/tutorial/write-scripts-for-the-mongo-shell/).

### Persistency

Persistence behaviour depends on "[write concern](http://docs.mongodb.org/manual/core/write-concern/)" setting (on a query time!) and this is a trade-off between latency and ability to recover.

[By default](http://docs.mongodb.org/manual/release-notes/drivers-write-concern/#driver-write-concern-change) operation confirmed when [mongod](http://docs.mongodb.org/manual/reference/program/mongod/#bin.mongod) (Primary) confirms that it received the write operation and applied the change to the in-memory view of data

To ensure that MongoDB can recover the data following a shutdown or power interruption, **journaled write concern** (acknowledges the write operation only after committing the data to the [journal](http://docs.mongodb.org/manual/reference/glossary/#term-journal)) or **replica acknowledged write concern (**write operation propagated to additional members of the replica set) may be used.

Starting MongoDB 3.0 compression is supported by the storage engine.

### Query Language, Entry Processor, User Defined Functions and Bulk Writes

Accordingly to [MongoDB Architecture Guide](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Architecture_Guide.pdf) MongoDB support 

* reach query language (KV-queries, range queries, text search) with query optimizations

* aggregation framework for group-by-like queries 

* MapReduce queries on JavaScript

* [bulk writes](http://docs.mongodb.org/v3.0/core/bulk-write-operations/)

### Indexes (local and global)

MongoDB [supports set of indexes](http://docs.mongodb.org/manual/core/index-types/): unique, compound, array, TTL, sparse, text).

Indexes are local, every shard will have its own index (containing just the documents in this shard).  In case of request, which affects few shards, they will be accessed in parallel (every shard reads its own local index shard) and then results merged.

Global indexes are not supported.

### Traffic encryption and access control

[Transport encryption is available via TLS/SSL](http://docs.mongodb.org/manual/tutorial/configure-ssl/) starting 3.0. Certain distributions of MongoDB do not contain support for SSL.

[Encryption at rest](http://docs.mongodb.org/master/core/security-introduction/#encryption-at-rest) is not available.

[Role-Based Access Control](http://docs.mongodb.org/manual/tutorial/manage-users-and-roles/) is available.

MongoDB Enterprise provides support for authentication using SASL and LDAP with [ActiveDirectory](https://docs.mongodb.org/v3.0/tutorial/configure-ldap-sasl-activedirectory/) and [OpenLDAP](https://docs.mongodb.org/v3.0/tutorial/configure-ldap-sasl-openldap/) plus authentication with [Kerberos](https://docs.mongodb.org/v3.0/tutorial/control-access-to-mongodb-with-kerberos-authentication/).

### Business model and commercial support

MongoDB, Inc distributed two versions of MongoDB, commercial version is named Enterprise MongoDB and includes:

* support

* [OpsManager](https://www.mongodb.com/products/ops-manager) product

* advanced security (Kerberos and LDAP etc.)

* commercial license

* platform certification

### Usage experience

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

## Cassandra

Free and Open-source database, written in Java. Datastax make business mostly on a support.

Used by many clients, Netflix shown benchmarks with "[Over a million writes per second on AWS](http://techblog.netflix.com/2011/11/benchmarking-cassandra-scalability-on.html)" (see also [presentation](http://www.slideshare.net/adrianco/cassandra-performance-on-aws))

**Wide-column** database (group of columns associated with the single key) based on [Log-Structured Merge-Tree](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.44.2782). [Datafiles are organized as SSTable](http://teddyma.gitbooks.io/learncassandra/content/model/where_is_data_stored.html) (SSTable is a simple abstraction to efficiently store large numbers of key-value pairs while optimizing for high throughput, sequential read/write workloads, so key cache, row cache and indexes are used).

Starting CQL3 require columns to be declared before used, so Cassandra is not anymore "schemaless", however [using a Map on Cassandra 2.1 we can store unstructured data](http://stackoverflow.com/questions/25098451/cql3-each-row-to-have-its-own-schema).

**JSON** datatype is not supported as schemaless and [feature request was rejected](https://issues.apache.org/jira/browse/CASSANDRA-6833). 

Nevertheless, [starting v 2.2](https://issues.apache.org/jira/browse/CASSANDRA-7970) JSON is [supported as input/output](http://www.datastax.com/dev/blog/whats-new-in-cassandra-2-2-json-support) format in [CQL](http://cassandra.apache.org/doc/cql3/CQL-2.2.html#insertJson). This may lead to conclusion, that since 01/Apr/15 Cassandra become **document-oriented**-like DB with **fixed document schema** (need to be confirmed).

### Architecture

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

### Data consistency model

**Strong** to **eventual** consistency depending on configuration ([tunable consistency](http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html)), more strong consistency will affect latency.

Cluster is **strong consistent** until number of nodes, required to read + nodes, required to write > replication factor. So QUORUM for read/write will guarantee **strong consistency**.

Cassandra chose **not** to implement vector clocks for performance reasons. Vclocks (typically) require a read before each write. By using last-write-wins in all cases, and ignoring the causality graph, Cassandra can cut the number of round trips required for a write and obtain a significant speedup. [The downside is that there is no safe way to modify a Cassandra cell, last write wins](https://aphyr.com/posts/294-call-me-maybe-cassandra/).

See also [Cassandra Consistency Features](https://quabase.sei.cmu.edu/mediawiki/index.php/Cassandra_Consistency_Features).

### ACID guarantees

Accordingly to [documentation](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_about_transactions_c.html) Cassandra does not use RDBMS ACID transactions with rollback or locking mechanisms, but instead offers atomic, isolated, and [durable transactions](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_ltwt_transaction_c.html) with eventual/tunable consistency that lets the user decide how strong or eventual they want each transaction’s consistency to be.

As a non-relational database, Cassandra does not support joins or foreign keys, and consequently does not offer consistency in the ACID sense. For example, when moving money from account A to B the total in the accounts does not change. Cassandra supports atomicity and isolation **at the row-level**, but trades transactional isolation and atomicity for high availability and fast write performance. Cassandra writes are durable.

[Distributed lightweight transactions](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_ltwt_transaction_c.html) are based on Paxos protocol.

So ACID guarantees of Cassandra are next

**Atomicity**: write is atomic at row-level; doesn’t rollback in case of fail on some replicas

**Consistency**: tunable with CL requirement (CP vs AP)

**Isolation**: row-level

**Durability**: durable, but persistence is asynchronous by default (tunable)

Also lightweight transactions available since Cassandra 2.0 for INSERT and UPDATE

### Replication schema

Cassandra stores few copies (replicas) of every partition, number of copies depends on replication factor. All replicas are equally important; there is no primary or master replica. 

Usage of replicas described in architecture section.

[While replication two strategies available](http://www.slideshare.net/DataStax/understanding-data-partitioning-and-replication-in-apache-cassandra): **simple strategy** (without considering rack or datacenter location) and **network topology** (more control over where replica rows are placed)

Also there is a [hinted handoff for cases, when replicas are offline, but write consistency met](http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_about_hh_c.html). Hinted handoff optimizes the cluster consistency process and anti-entropy when a replica-owning node is not available, due to network issues or other problems, to accept a replica from a successful write operation. Hinted handoff is not a process that guarantees successful write operations, [except when a client application uses a consistency level of ANY](http://www.slideshare.net/DataStax/understanding-data-consistency-in-apache-cassandra)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_5.png)

### Sharding Schema

Data is partitioned in Cassandra across participating nodes based on column family row key. 

Two basic strategies: 

* Random partitioning (default and recommended). MD5 hash from every column family row key used

* Ordered partitioning (ByteOrdered and OrderPreserving). Stores column family row keys in sorted order. Not recommended because of this limitation and because globally ordering all your partitions generates hot-spots.

### CAP-theorem

Cassandra is typically classified as an AP system, meaning that availability and partition tolerance are generally considered to be more important than consistency in Cassandra. But Cassandra can be tuned with replication factor and consistency level to also meet CP.

### Split-brain and recovery from a split brain

Cassandra may be vulnerable to split-brain if configured as AP-system.

Recovery from split-brain include schema and data merge. 

Data merged by "last-wins" principle. Schema disagreement [may require manual recovery](http://wiki.apache.org/cassandra/LiveSchemaUpdates)   

### Rack-aware replication and cross-datacenter replication

[Accordingly to documentation](http://docs.datastax.com/en/cassandra/2.0/cassandra/architecture/architectureDataDistributeReplication_c.html), Cassandra supports rack-aware and datacenter-aware replication (multi-datacenter replication). "Last wins" merge policy used.

From experience, multi-datacenter replication was used under heavy load for years and was pretty stable.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_6.png)

### Fault-tolerance

High-availability and fault tolerance is built on replication and [peer-to-peer architecture to survive replica fail](http://www.datastax.com/dev/blog/how-cassandra-deals-with-replica-failure)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_7.png)

Fault-tolerance (how many nodes may be lost) depends on 

* replication factor

* consistency level used for read and writes

* network topology settings

In general, Cassandra will survive loss of (REPLICATION FACTOR - CONSISTENCY LEVEL NODES COUNT) nodes

### Dynamic scalability

[Cassandra is dynamically scalable](http://docs.datastax.com/en/cassandra/2.0/cassandra/operations/opsAddingRemovingNodeTOC.html) (add/remove nodes and datacenters). [Network topology change may require downtime of specific node or full cluster repair.](http://docs.datastax.com/en/cassandra/2.0/cassandra/operations/opsMoveNodeRack.html)

Cassandra automatically rebalance data if new node added, but d[oes not automatically remove data from nodes that lose part of their partition range to a newly added node](http://docs.datastax.com/en/cassandra/2.0/cassandra/tools/toolsCleanup.html) and manual cleanup required.

### Persistency

Cassandra is using [due to log-structured merge-tree](https://github.com/wiredtiger/wiredtiger/wiki/LSMTrees) for high write throughput, so Cassandra is [mostly write-oriented](https://github.com/wiredtiger/wiredtiger/wiki/Btree-vs-LSM). In the same time, read throughput and latency are good enough.

Cassandra persist data accordingly to durability settings, but synchronous persistency is slow. 

In default mode write confirmed when node added change to memory and distributed over the cluster, commit log will be dumped to disk **by timeout**. Cassandra writes are first [written to the CommitLog](https://wiki.apache.org/cassandra/Durability), and then to a [per-ColumnFamily structure called a Memtable](http://codrspace.com/b441berith/cassandra-sstable-memtable-inside/). When a Memtable is full, [it is written to disk as an SSTable](https://wiki.apache.org/cassandra/MemtableSSTable). Also there is [compaction process](http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_write_path_c.html).

#### Write parh

When a [write occurs](http://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_write_path_c.html), Cassandra stores the data in a structure in memory, the memtable, and also appends writes to the commit log on disk (**dumped asynchronously by default**), providing configurable durability. The commit log receives every write made to a Cassandra node, and these durable writes survive permanently even after power failure (but depends on settings, asynchronous by default). 

The memtable is a **write-back cache** of data partitions that Cassandra looks up by key. The memtable stores writes until reaching a limit (configurable), and then is flushed. replay time. 

To flush the data, Cassandra sorts memtables by token and then writes the data to disk sequentially.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_8.png)

#### Read path

When a [read request for a row comes in to a node](http://docs.datastax.com/en/cassandra/1.2/cassandra/dml/dml_about_read_path_c.html), the row must be combined from all SSTables on that node that contain columns from the row in question, as well as from any unflushed memtables, to produce the requested data.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_9.png)

### Query Language, Entry Processor, User Defined Functions and Bulk Writes

CQL query language available without analytical functionality.

There is no Entry Processor in this DB, but there are triggers. In the same time, 1 year ago Datastax engineers unofficially didn’t recommended to not use them due to feature is new and maybe unstable.

### Indexes (local and global)

[Secondary indexes](http://docs.datastax.com/en//cql/3.1/cql/ddl/ddl_when_use_index_c.html) [are supported](https://issues.apache.org/jira/browse/CASSANDRA-3680). Indexes are local (each node only indexes data that it holds locally) and distributed over the cluster, so for search request will be sent to multiple nodes in parallel. 

Also this means that you must have CL nodes available for all token ranges in the cluster in order to complete a query regardless of cluster size (for example, with RF = 3, when two out of three consecutive nodes in the ring are unavailable, all secondary index queries at CL = QUORUM will fail, however secondary index queries at CL = ONE will succeed)

The row and index updates are one, atomic operation.

### Traffic encryption and access control

[Node-to-node](http://docs.datastax.com/en/cassandra/1.2/cassandra/security/secureSSLNodeToNode_t.html) and [client-to-node](http://docs.datastax.com/en/cassandra/1.2/cassandra/security/secureSSLClientToNode_t.html) encryption of data transferred between nodes, including gossip communication available with using SSL

[Internal authentication](http://docs.datastax.com/en/cassandra/1.2/cassandra/security/secureInternalAuthenticationTOC.html) and [internal authorization](http://docs.datastax.com/en/cassandra/1.2/cassandra/security/secureInternalAuthorizationTOC.html) with object permissions is available.

[LDAP and Kerberos integration](http://docs.datastax.com/en/datastax_enterprise/4.6/datastax_enterprise/sec/secTOC.html) is available with [DataStax Enterprise](http://docs.datastax.com/en/datastax_enterprise/4.6/datastax_enterprise/newFeatures.html).

### Caching

Starting Cassandra 2.0 [supports off-heap storages](http://www.datastax.com/dev/blog/off-heap-memtables-in-Cassandra-2-1) for [bloom filters, compression offsets map and partition summary](http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_off_heap_c.html). Also [row cache may be placed off-heap](http://docs.datastax.com/en/cassandra/2.0/cassandra/operations/ops_configuring_caches_c.html), but this will lead to de-serialization overhead. Key cache is on-heap.

### Business model and commercial support

DataStax Enterprise is [available by subscription](http://www.datastax.com/what-we-offer/products-services/production).

### Usage experience

Cassandra was used as storage (16 nodes, cross-datacenter replication), our experience is next:

* good horizontal scalability with automatic sharding.

* stable work on 2000 - 3000 tps with stable master-master cross-datacenter replication

* CQL is not so advanced as SQL, no joins, group by, select...where on arbitrary columns and analytical functions

* no table statistic, so select count is expensive (full scan), so if we need statistic - need to support statistic table.

* Cassandra may be affected by GC sometimes, so if SLA, for example, is strongly >99% of <10ms for request time, then Cassandra should not be used. 

* there is no performance gain from key and row caches if dataset fits memory (this is obvious)

* query engine may be memory intensive on complicated queries (select count limit 50 000 will lead to allocating of result set for 50 000 rows, which is unnecessary)

* two years of reliable work in production without significant issues

* stable cross-datacenter replication

* used as a Catalog storage instead of Coherence to simplify deployment architecture by eliminating primary-secondary schema for fast recover without Coherence persistency usage. Cassandra is very fast when dataset is in memory (1-2 msec read).

* By our experience, Cassandra is good as reliable records storage with high-throughput, horizontal scalability and reliable cross-datacenter replication, if latency > 20-30ms but less than 50ms average acceptable

## HBase

Key-value NoSQL DB build on top of HDFS (distributed file system). Written on Java, distributed with the most of Hadoop distributions.

HBase is also **wide-column** and similar file format and architecture as Cassandra. HBase supports a "bytes-in/bytes-out".

### Architecture

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

### Data consistency model

Due to Region Master HBAse support **strong consistency** at the level of a single row.

[In fact HBase ](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.3/bk_system-admin-guide/content/sysadminguides_ha-HBase-data-consistency.html)[guarantees timeline consistency](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.3/bk_system-admin-guide/content/sysadminguides_ha-HBase-data-consistency.html) for all data served from Region Servers in secondary mode, meaning all HBase clients see the same data in the same order, but that data may be slightly stale. Only the primary Region Server is guaranteed to have the latest data. 

Timeline consistency simplifies the programming logic for complex HBase queries and provides lower latency than quorum-based consistency

### ACID guarantees

[HBase supports ACID in limited ways](http://hadoop-hbase.blogspot.com/2012/03/acid-in-hbase.html), namely Puts to the same row provide all ACID guarantees. ([HBASE-3584](https://issues.apache.org/jira/browse/HBASE-3584) adds multi op transactions and [HBASE-5229](https://issues.apache.org/jira/browse/HBASE-5229) adds multi row transactions, but the principle remains the same)

### Replication schema

[Replication is based on HDFS ](http://hortonworks.com/blog/introduction-to-hbase-mean-time-to-recover-mttr/)functionality.

Data written in HDFS is replicated on several nodes:

* HBase writes the data in HFiles, stored in HDFS. HDFS replicates the blocks of these files, by default 3 times.

* HBase uses a commit log (or Write-Ahead-Log, WAL), and this commit log is as well written in HDFS, and as well replicated, again 3 times by default.

### Sharding Schema

Sharding is configurable by splitting on regions. Initially, [HBase creates only one region per table](http://hortonworks.com/blog/apache-hbase-region-splitting-and-merging/). Once a region gets to a certain limit, it is automatically split into two regions. HBase also enables clients to force split an online table from the client side.

Master includes load balancing between origin servers and automatic re-balancing of shards. 

Master keep information about table to region split and region to origin mapping (Zookeeper used). 

Client download meta-information from master until region is missed, then will re-download (when origin server is unavailable or region unavailable on origin server)

### CAP-theorem

HBase is CP-system (HDFS).

### Split-brain and recovery from a split brain

Not affected to split-brain because it’s CP.

### Rack-aware replication and cross-datacenter replication

Replication in [HDFS is rack-aware](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html).

Cluster to cluster replication can be defined at the column-family level, works in the background, and keeps all edits in sync between clusters in the replication chain. Replicationis based on distribution of WAL.

Cluster-to-cluster replication has three modes: master-slave, master-master, and cyclic.

### Fault-tolerance

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

### Dynamic scalability

[Adding of a new Regions Server](http://hbase.apache.org/book.html#adding.new.node) doesn’t require downtime, but [Region Server decommission](http://hbase.apache.org/book.html#decommission) may lead to Region temporary unavailability 

### Persistence

Persistence is based on HDFS and similar to Cassandra (write-ahead-log with memstore which is flushed asynchronously)

Good overview of the read-write paths can be found in this presentation: [Storage Systems for big data - HDFS, HBase, and intro to KV Store - Redis ](http://www.slideshare.net/sameertiwari33/storage-for-big-data-30957881)

The main concern is a durability of a WAL log, need to confirm that [WAL flush](http://blog.cloudera.com/blog/2012/06/hbase-write-path/) is synchronous.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_11.png)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_12.png)

### Query Language, Entry Processor, User Defined Functions and Bulk Writes

HBase have kind of EntryProcessors (CoProcessors) which behaves like triggers, nevertheless coprocessors are not designed to be used by end users of HBase, but by HBase developers who need to add specialized functionality to HBase. 

No Query Language for HBase, but there is 3rd party project, Phoenix, QL for HBase. 

Also [Phoenix project provides set of indexes, including global indexes](https://phoenix.apache.org/secondary_indexing.html) for read heavy, low writes cases

Bulk writes are supported.

### Indexes (local and global)

As well as in Cassandra, [indexes are local (region-level)](http://www.z2-environment.net/blog/2014/01/secondary-indexes-in-hbase/) and integrated with HBase’s Write-Ahead-Log (WAL), indexes are written directly and atomically for data within the region server local regions to the region server.

So, every region server holds indexes for its data. This approach is implemented by HIndex and IHBase. This may affects scalability for broad queries. 

For global indexes (covering indexes) see Phoenix

### Traffic encryption and access control

Traffic encryption, data encryption and access control based on Hadoop security (SSL, LDAP and RBAC)

### Caching schema

In the default configuration, [HBase uses a single on-heap cache](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/admin_hbase_blockcache_configure.html). Off-heap [BucketCache](http://hortonworks.com/blog/hbase-blockcache-101/) can be configured and will be used for [Bloom filters](https://en.wikipedia.org/wiki/Bloom_filter), indexes and cache data blocks.

### Business model and commercial support

HBase is the part of major Hadoop distributions and supported by distribution vendors

### Usage experience

HBase was used by GridDynamics on one of the projects to store and analyze clickstream few years, results are next:

* stable enough for production usage

* CLI interface is not rich and just get\put    

### Recommendations

HBase is good to use with Hadoop, utilizing existing Hadoop deployment. HBase will provide random access instead of sequential to Hadoop plus HBase uses existing Hadoop architecture

HBase better for Hadoop then remote storage (Cassandra etc.) because you don’t need to make long network call to remote system and transfer data. In case of HBase usage, local map task may use region stored on the same node. Also there is extension for MR to read/parse HBase files (HFileInputFormat), so analytic over HBAse is extremely fast.

Usage of HBase as a pure NoSQL solution raises two concerns:

* pipeline of request processing is simply longer than in Cassandra and requires more work, see read path. Nevertheless [was a posts which said that HBase may be fast as Cassandra](http://www.slideshare.net/HBaseCon/features-session-5). 

* HBase relies on Hadoop architecture and deployment, so it’s a cheap and simple solution if used with Hadoop, but expensive (HDFS, Zookeeper) if used as pure NoSQL storage.

## Redis

Open-source (BSD license ) [key-value cache and store](http://redis.io/documentation), written in C. [Single-thread](http://redis.io/topics/clients) and [memory-optimized](http://redis.io/topics/memory-optimization).

[Used by](http://redis.io/topics/whos-using-redis) Twitter, GitHub,  Pinterest, Craigslist, Digg, StackOverflow, Flickr.

[Value typed, base types supported](http://redis.io/topics/data-types-intro): [binary-safe strings, lists, sets, sorted sets, maps, bit arrays and HyperLolLogs](http://redis.io/topics/data-types) (probabilistic data structure used in order to count unique things, technically this is referred to estimating the cardinality of a set). 

Value typed until record exists. JSON is **not** supported.

[Keyspace notifications](http://redis.io/topics/notifications) via [Pub/Sub](http://redis.io/topics/pubsub) and Pub/Sub available.

### Architecture

There are three different architecture which is possible in Redis: single partition with replication (optional), Sentinel HA and Redis Cluster. 

Due to asynchronous replication and and persistence, Redis will not prevent data loss in any of those schemas, just will minimize it.

#### Single Partition with Replication (optional)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_13.png)

##### Replication

Replication is [based on asynchronous pre-configured master-slave replication](http://redis.io/topics/replication).

Replicas may serve read requests for scalability, but due to asynchronous replication data may be stale. Master is able to serve read-write. 

Starting 2.8 Redis sent only difference while replication.

Starting with Redis 2.8, it is possible to configure a Redis master to accept write queries only if at least N slaves are currently connected to the master. Detection is asynchronous.

**Also automatic restart of the master without persistency will lead to erasing of the replica’s storages.**

##### Sharding

[Sharding (partitioning) is implemented in a 3 different ways on the different parts of a software stack](http://redis.io/topics/partitioning): client side partitioning, proxy assisted partitioning and query routing which will be overviewed in a Redis Cluster.

Some features of Redis will not be available with partitioning:

* Operations involving multiple keys are usually not supported. For instance you can't perform the intersection between two sets if they are stored in keys that are mapped to different Redis instances.

* Redis transactions involving multiple keys can not be used.

* The partitioning granularity is the key, so it is not possible to shard a dataset with a single huge key like a very big sorted set.

* When partitioning is used, data handling is more complex, for instance you have to handle multiple RDB / AOF files, and to make a backup of your data you need to aggregate the persistence files from multiple instances and hosts.

* Adding and removing capacity can be complex. For instance Redis Cluster supports mostly transparent rebalancing of data with the ability to add and remove nodes at runtime, but other systems like client side partitioning and proxies don't support this feature. However a technique called Pre-sharding helps in this regard.

###### Client side partitioning

Client side partitioning means that the clients directly select the right node where to write or read a given key. Many Redis clients implement client side partitioning.

[TODO add schema]

###### Proxy assisted partitioning

Proxy assisted partitioning means that our clients send requests to a proxy that is able to speak the Redis protocol, instead of sending requests directly to the right Redis instance. The proxy will make sure to forward our request to the right Redis instance accordingly to the configured partitioning schema, and will send the replies back to the client. The Redis and Memcached proxy [Twemproxy](https://github.com/twitter/twemproxy) implements proxy assisted partitioning.

[TODO add schema]

##### Fault-tolerance

There is no High Availability in this case and automatic failover.

#### Sentinel High Availability

[Redis Sentinel](http://redis.io/topics/sentinel) provides high availability abilities for Redis:

* automatic failover

* monitoring

* notifications

* configuration provide (service discovery for clients)

Sentinel is a distributed system of a Sentinel processes with auto-discovery.

The sum of Sentinels, Redis instances (masters and slaves) and clients connecting to Sentinel and Redis, are also a larger distributed system with specific properties.

[TODO add schema]

Quorum (configurable) of Sentinel processes may decide that master is dead and elect new master from slaves informing clients. Client **need to support Sentinel**.

##### Replication

The same as for single master schema, but in case of failover slave may become a master. In this case part of the changes [will be lost](https://aphyr.com/posts/283-call-me-maybe-redis) due to **eventual consistency**. 

##### Sharding 

The same as for single master schema.

##### Fault Tolerance

Will survive fail of master, Sentinel process and slaves until there are enough replicas accessed and Sentinel quorum.

#### Redis Cluster

Query routing means that you can send your query to a random instance, and the instance will make sure to forward your query to the right node. 

[Redis Cluster](http://redis.io/topics/cluster-tutorial) implements an hybrid form of query routing, with the help of the client (the request is not directly forwarded from a Redis instance to another, but the client gets redirected to the right node).

Redis Cluster is the preferred way to get **automatic sharding** and **high availability**. It is generally available and production-ready as of [April 1st, 2015](https://groups.google.com/d/msg/redis-db/dO0bFyD_THQ/Uoo2GjIx6qgJ)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_14.png)

[TODO Rewrite schema]

##### Sharding

Redis cluster divides keyspace onto 16384 hash slots (CRC 16 mod 16384) and assign range to every node, so every node in a Redis Cluster is responsible of a subset of the hash slots. Move of the hash slot is automatic.

Redis Cluster supports multiple key operations as long as all the keys involved into a single command execution (or whole transaction, or Lua script execution) all belong to the same hash slot. The user can force multiple keys to be part of the same hash slot by using a concept called **hash tags**.

##### Replication

In order to remain available when a subset of master nodes are failing or are not able to communicate with the majority of nodes, [Redis Cluster uses a master-slave model](http://redis.io/topics/cluster-spec) where every hash slot has from 1 (the master itself) to N replicas (N-1 additional slaves nodes)

Master and slave roles are pre-configured, also this is possible to configure replica to serve particular master.

As well as for Sentinel, slave may become a master. In this case part of the changes will be lost due to **eventual consistency**. 

In a Redis Cluster replica may be assigned to the master automatically and automatically migrated.

##### Fault Tolerance

In case of master fail, slave will be elected as a new master. If master and slaves failed, Redis cluster will not be able to continue serving of the partition.

Split-brain is prevented by majority principle.

### Data consistency model

Until fail this will be a **strong (linear, due to single master and single thread) view consistency** for master and **eventual consistency** for replicas. In case of failover - **eventual consistency** with data loss.

### ACID guarantees

Durability is not guaranteed.

### CAP-theorem

CP for single master.

As for cluster this is depends on implementation. For Redis Cluster can be considered as AP until split discovery and CP after (transition with data loss).

### Split-brain and recovery from a split brain

Split brain is prevented by Sentinel and Redis Cluster by majority.

### Rack-aware replication and cross-datacenter replication

There is no rack-aware replication inside Redis, but [Redis Labs](https://en.wikipedia.org/wiki/Redis_Labs) declared [rack-awareness](https://redislabs.com/redis-enterprise-documentation/rack-zone-awareness) (as well as HA) in their product, [Redis Labs Enterprise Cluster](https://redislabs.com/redis-enterprise-documentation/overview), which is based on Redis.

### Dynamic scalability

Supported for replicas, Sentinel and Redis Cluster

### Persistence

Redis provides [asynchronous persistence](http://redis.io/topics/persistence) to files:

* Snapshot ([RDB](https://github.com/sripathikrishnan/redis-rdb-tools/wiki/Redis-RDB-Dump-File-Format))

* AOF (Append-only file) operation log

**RDB file** is a compact snapshot of the storage which is created automatically and good for backup, disaster recovery and quick restart. 

Save of snapshot may be configured by timeout or by changes count. 

In the same time, creating of the snapshot is a heavy operation (full dump) which should not be invoked too frequently, so in case of node fail, RDF can be old.

**AOF** is a command log of every write operation received by the server, to be replayed on start-up, reconstructing original dataset. 

Commands are logged using the same format as the Redis protocol itself, in an append-only fashion. 

Redis is able to rewrite the log on background when it gets too big. Nevertheless this means that re-play may take significantly more time than load from RDF.

Also in case of failure, AOF file may be incomplete and data after last [filesync](http://linux.die.net/man/2/fsync) may be damaged and ignored.

AOF is updated on every write operation, but file sync is triggered depending on the setting:

* fsync every time a new command is appended to the AOF. Very very slow, very safe.

* fsync every second. Fast enough (in 2.4 likely to be as fast as snapshotting), and means that you can lose 1 second of data if there is a disaster.

* Never fsync, just put your data in the hands of the Operating System. The faster and less safe method.

So, as well as in almost all NoSQL solutions, persistence is asynchronous (tunable, but dramatically affects performance). But in other NoSQL solution this is compensated by **synchronous replication**. 

**This means that Redis** [can’t guarantee durability under any cluster schemas](https://aphyr.com/posts/283-call-me-maybe-redis) (Sentinel or cluster).

### Query Language, Entry Processor, User Defined Functions and Bulk Writes

Query Language is not available, [system of commands instead](http://redis.io/commands), nevertheless only basic functionality for records (get/put, increment and queue operations) available.

- Is Query Language and bulk writes available

- Is Entry Processor and User Defined Functions supported

- Is Aggregation functionality available

Also available [User Defined Functions on LUA which can be evaluated on the node](http://redis.io/commands/eval), which is a good replacement for bulk operations in some cases.

[Mass upload](http://redis.io/topics/mass-insert) and [pipelining](http://redis.io/topics/pipelining) (bulk operations) is available. Also [transactions](http://redis.io/topics/transactions) (batch) may be used

### Indexes (local and global)

There is no indexes, Redis is a pure key-value storage with access by key only.

### Traffic encryption and access control

[Redis does not support encryption](http://redis.io/topics/encryption). In order to implement setups where trusted parties can access a Redis instance over the internet or other untrusted networks, an additional layer of protection should be implemented, such as an SSL proxy or [Slipped](http://www.tarsnap.com/spiped.html).

[Redis is designed to be accessed by trusted clients inside trusted environments](http://redis.io/topics/security). This means that usually it is not a good idea to expose the Redis instance directly to the internet.

In the same time Redis:

* can bind specific interface

* have tiny layer of authentication (password, pre-defined in **redis.conf** configured file)

* can be configured to disable or rename particular commands in **redis.conf**

### Caching schema

Redis is a pure in-memory storage with persistence and replication. Also can work as [LRU cache](http://redis.io/topics/lru-cache).

Records [may have expiration setting](http://redis.io/commands/expire).

### Business model and commercial support

[Community Support](http://redis.io/support).

Commercial Support available for [Managed Redis](https://redislabs.com/) instances from [Redis Labs](http://redislabs.com/) (the official sponsor of the Redis Project).

### Usage experience

Results are next:

* Extremely simple configuration and [very fast](http://redis.io/topics/latency-monitor), [hundreds thousands tps](http://redis.io/topics/benchmarks).

* Used as a cache for auth token management, for password bruteforce protection (counters in Redis)

* Redis Sentinel used for master-slave replication.

* No issues for the last year when used as a cache

## Aerospike

Flash-optimized in-memory [wide-column database with schema-less data model](http://www.aerospike.com/docs/guide/kvs.html) with [strong typization](http://www.aerospike.com/docs/guide/data-types.html) and [large collections](http://www.aerospike.com/docs/guide/ldt.html). Written in C.

JSON is not supported, [the only way to store JSON is a string or decomposition to List/Map](https://github.com/aerospike/complex-data-types).

### Architecture

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

### Data consistency model

Declared **strong consistency** due to master replica and synchronous replication until split-brain.

[Also declared that](http://www.aerospike.com/docs/architecture/assets/AerospikeACIDSupport.pdf) "So far, Aerospike has focused on continuous availability and provides the best possible consistency in an AP system by using techniques to avoid partitions (**SIC!**)"

### ACID guarantees

Aerospike [declared ACID guarantees](https://www.aerospike.com/docs/architecture/assets/AerospikeACIDSupport.pdf) and high consistency.

Late [analyze from Jepsen](https://aphyr.com/posts/324-call-me-maybe-aerospike) was published with demonstration of the case when, Aerospike may lost data: "its strongest guarantees are similar to Cassandra or Riak in Last-Write-Wins mode. It may be a safe store for immutable data, but updates to a record can be silently discarded in the event of network disruption. Because Aerospike’s timeouts are so aggressive–on the order of milliseconds–even small network hiccups are sufficient to trigger data loss."

After this publication, Aerospike said that they "[apologize for overstepping the bounds with our marketing pitch and not presenting our consistency claims within the proper technical context](https://discuss.aerospike.com/t/aerospike-loses-data/1250/3)" on their forum.

### Replication schema

For reliability, Aerospike replicates partitions on one or more nodes. One node becomes the data master **for reads and writes** for a partition and other nodes store replicas.

When a node receives a write request, it saves the data AND it forwards the write request to the replica node. Once the replica node confirms that the data has been written successfully AND the node has written the data itself, then confirmation is sent to the client that the write operation was successful. So write to all replicas is synchronous.

The replication factor is a configuration parameter. More replicas mean more reliability, but higher demand on the cluster as write requests must go to all copies of the data.

### Sharding Schema

[Sharding schema is similar to Cassandra with tunable replication factor](http://www.aerospike.com/docs/architecture/data-distribution.html)*.*

Data is distributed evenly across nodes in a cluster using the Aerospike Smart Partitions™ algorithm. Declared "extremely random hash function ensures that partitions are distributed very evenly within a 1-2% error margin".

To determine where a record should go, the record key (of any size) is hashed into a 20-byte fixed length string using RIPEMD160, and the first 12 bits form a partition ID which determines which of the partitions should contain this record. 

Automatic sharding and re-balancing.

### CAP-theorem

[Accordingly to whitepaper](http://pages.aerospike.com/rs/aerospike/images/Acid_Whitepaper.pdf), "In the presence of failures, the cluster can run in one of two modes – AP (available and partition tolerant) or CP (consistent and partition tolerant)", **which is a bit strange** because in the next sentence they states that “In the future, Aerospike will add support for CP mode as an additional deployment option.”. Also in the same paper they said that “Aerospike does not yet support CP mode configuration.”

Accordingly to the same whitepaper, they declared CAP in the same time (**SIC!**), "High consistency on AP mode" **which is ridiculous**. Inside the topic, they explained that “High consistency on AP mode” means “that a replicated Aerospike cluster provides high consistency and high availability during node failures and restarts so long as the cluster does not split into separate partitions” (**SIC!**), so really this is CA. Which is also strange in the light of this discussion “[You Can’t Sacrifice Partition Tolerance](http://codahale.com/you-cant-sacrifice-partition-tolerance/)”

Looks like this is one more item which may lead to "apologize for overstepping the bounds with our marketing pitch" 

### Split-brain and recovery from a split brain

Allows splitbrain state, so availability over consistency, AP from CAP

Two merge policies **may be followed**. Either Aerospike will auto-merge the two data items (default

behavior today) or keep both copies for application to merge later (future) (**SIC!** [Accordingly to an Aerospike forum this feature is not available yet](https://discuss.aerospike.com/t/conflict-resolution-handle-by-application-option-available/1228)).

Auto merge works as follows:

*  TTL (time-to-live) based: The record with the highest TTL wins

* Generation based: The record with the highest generation wins

Application merge works as follows:

*  When two versions of the same data item are available in the cluster, a read of this value will return both versions, allowing the application to resolve the inconsistency.

* The client application – the only entity with knowledge of how to resolve these differences – must then re-write the data in a consistent fashion.

### Rack-aware replication and cross-datacenter replication

[Asynchronous Master-to-Master cross datacenter replication](http://www.aerospike.com/docs/architecture/xdr.html) in [Enterprise Edition](http://www.aerospike.com/products-services/). Also [rack-aware replication supported](http://www.aerospike.com/docs/architecture/rack-aware.html).

Currently [XDR (Cross-DataCenter Replication) is not compatible with Security](http://www.aerospike.com/docs/guide/security.html) (audit and role based access) turned on.

### Fault-tolerance

High-availability and fault tolerance is built on replication and peer-to-peer architecture to survive replica fail. 

The remaining nodes automatically perform data migration to copy the replica partitions to new data masters.

So fault-tolerance depends on number of replica.

### Dynamic scalability

Need to be confirmed.

### Persistence

Declared [Hybrid Storage concept](http://www.aerospike.com/docs/architecture/storage.html), Indexes stored in memory, data on SSD with caches in RAM.

### Query Language, Entry Processor, User Defined Functions and Bulk Writes

Own query language, [AQL](http://www.aerospike.com/docs/tools/aql/), [query-level aggregations are available via UDF](http://www.aerospike.com/docs/guide/aggregation.html)

[Aggregations](http://www.aerospike.com/docs/architecture/secondary-index.html#aggregations) are available via Aggregation Framework (Query records feed to aggregation framework to perform filters, aggregations, etc. On each node, the query result is sent to the UDF sub-system to begin processing the results as a stream of records. The results from each node are then collected by the client application, which may also perform additional operations on the data.)

[UDF on LUA](http://www.aerospike.com/docs/architecture/udf.html) acts as a triggers and executed in the main flow of the transaction. Also may operate on a stream (see aggregations) for MapReduce-like operations.

### Indexes (local and global)

[Primary indexes are in-memory only](http://www.aerospike.com/docs/architecture/primary-index.html), so when aerospike server starts, it walks through data on the storage and creates primary index for all the data partitions, but index may survive fast restart.

[Secondary indexes](http://www.aerospike.com/docs/architecture/secondary-index.html) are local. The query to retrieve results from the secondary index is sent to every cluster node, so distributed query will be used. [Secondary indexes will not survive fast restart](http://www.aerospike.com/docs/guide/query.html).

### Traffic encryption and access control

Aerospike does [not provide encryption of data across the network](http://www.aerospike.com/docs/operations/plan/network/)

Audit and role based access is not available for XDR configuration.

### Business model and commercial support

Community edition is open-source, Enterprise edition is closed-source and available by subscription for data volume.

### Usage experience

We had no experience with this database. 

Colleagues reported easy installation and configuration

In the same time some "overstepping the bounds with our marketing pitch" (see ACID, CAP and split-brain sections) looks very alarming.

## CouchBase [TODO]

CouchBase is a wide-column database written on C++, so no GC, no heap and low latency with a lot of data, forked from CouchDB (??)

### Architecture

CouchBase is like Cassandra by structure, by one of servers will be master for particular shard, so read and write to this shard will be routed to this shard. 

So this is strong consistency until master fail. Due to asynchronous sync of replicas, data may be failed (??, check configuration). New master election and re-balancing will occur on fail, also if asynchronous replication was used, we’ll get old data.

### Replication schema

Master-slave replics, write to master only, no scalability to write? 

Replication options (??), read from slave (??) synchronous syn to slave (??)

### Indexes

As well as in Cassandra, indexes are local (each node only indexes data that it holds locally) and distributed over the cluster, so for search request will be sent to multiple nodes in parallel. This affects scalability. Global indexes not supported.

### Query Language, Entry Processor, User Defined Functions and Bulk Writes

No Query Language (??)

Batches available (??)

Named views

### Rack-aware replication and cross-datacenter replication

Cross datacenter replication is free (DC-DC master-master), but rack-awareness (to distribute replicas over availability zones) is paid (available in CouchBase Enterprise)

### Caching Schema

CouchBase have fast caches and provides memcache protocol-based access

### Business model and commercial support

CouchBase is supported by commercial company

# Usecases

Cassandra, CouchBase and Aerospike good as System of Records and they are competitors in the same area.

Mongo is good for analytics and as document-oriented DB where HA is not critical

HBase is good to work with Hadoop

Redis is good for cache, where durability is not required

Coherence is a strong consistency system, so IO and network glitches may affect latency and throughput.

# Additional Materials

[Strong consistency models](https://aphyr.com/posts/313-strong-consistency-models)

[ACID](https://en.wikipedia.org/wiki/ACID)

[Difference between Linearizability and Serializability](http://stackoverflow.com/questions/4179587/difference-between-linearizability-and-serializability)

[What is eventual consistency?](http://www.quora.com/What-is-eventual-consistency)

[Top 5 Considerations When Evaluating NoSQL Databases](https://s3.amazonaws.com/info-mongodb-com/10gen_Top_5_NoSQL_Considerations.pdf)

[MongoDB Architecture Guide](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Architecture_Guide.pdf)

[Call me maybe: MongoDB](https://aphyr.com/posts/284-call-me-maybe-mongodb)

[Call me maybe: MongoDB stale reads](https://aphyr.com/posts/322-call-me-maybe-mongodb-stale-reads)

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

[Secondary indexes in HBase Phoenix](https://github.com/forcedotcom/phoenix/wiki/Secondary-Indexing)

[HBase : Secondary Indexes and Alternate Query Paths](http://hbase.apache.org/book.html#secondary.indexes)

[Apache HBase Reference Guide](http://hbase.apache.org/book.html)

[Secondary Indexes in HBase](http://www.z2-environment.net/blog/2014/01/secondary-indexes-in-hbase/#gti_refs)

[AeroSpike architecture](http://www.aerospike.com/docs/architecture/)

[AeroSpike data model](http://www.aerospike.com/docs/architecture/data-model.html)

[AeroSpike data management](http://www.aerospike.com/docs/architecture/data-management.html)

[AeroSpike FAQ](http://www.aerospike.com/docs/guide/FAQ.html)

[AeroSpike distribution](http://www.aerospike.com/docs/architecture/distribution.html)

[AeroSpike clustering](http://www.aerospike.com/docs/architecture/clustering.html)

[Base: An Acid Alternative](http://queue.acm.org/detail.cfm?id=1394128)

[MongoDB FAQ](https://www.mongodb.com/faq)

[MongoDB Multi-Data Center Deployments](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Multi_Data_Center.pdf)

[Cassandra background-and-architecture](http://www.slideshare.net/yellow7/cassandra-backgroundandarchitecture)

[GridDynamics Cooking Cassandra](http://www.slideshare.net/Open-IT/cassandra-meetup-april-2014) presentation

[Using Apache Cassandra: What is this thing, and how do I use it?](http://www.slideshare.net/jeremiahdjordan/cassandra-intro-27171544)

[NoSQL Essentials: Cassandra](http://www.slideshare.net/frodriguezolivera/nosql-essentials-cassandra-15625311)

[CAP : You Can’t Sacrifice Partition Tolerance](http://codahale.com/you-cant-sacrifice-partition-tolerance/)

[CAP : Please stop calling databases CP or AP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)

[CAP : CAP Twelve Years Later: How the "Rules" Have Changed](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed)

