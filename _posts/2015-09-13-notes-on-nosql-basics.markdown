---
layout: post
title:  "Notes on NoSQL : Basics and key differences between relational and NoSQL databases"
date:   2015-09-13 04:28:00
update: 2015-09-14 12:55:00
categories: NoSQL Architecture
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Principal Software Engineer and Grid Architect in San Francisco, USA. You can subscribe to my new posts in my <a href="http://sotnikdv.github.io">personal blog</a> or find me in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
editable: true
---

NoSQL is frequently named a brand-new and modern "silver bullet" for all challenges and usecases in a data storing and processing. Also it declared a comprehensive replacement for "outdated relational databases". Well, that's not true. 

In this article I'll try to explain why NoSQL technology was re-invented, which challenges returned this old technology back to life, strengths and weakness and usecases. Understanding of this is important to choose right technology for your case.

This is the one of the few articles in "Notes on NoSQL" sequence  
**[Notes on NoSQL : Basics and key differences between relational and NoSQL databases](/nosql/architecture/2015/09/13/notes-on-nosql-basics.html)**  
[Notes on NoSQL : MongoDB, unfulfilled hopes and possible bright future](/nosql/architecture/2015/09/13/notes-on-nosql-mongodb.html)  
[Notes on NoSQL : Years in production with Apache Cassandra](/nosql/architecture/2015/09/13/notes-on-nosql-cassandra.html)  
[Notes on NoSQL : Apache HBase, database on the top of Hadoop](/nosql/architecture/2015/09/13/notes-on-nosql-hbase.html)  
[Notes on NoSQL : Redis, pretty simple and jet fast cache](/nosql/architecture/2015/09/13/notes-on-nosql-redis.html)  
[Notes on NoSQL : Aerospike and risqué promises](/nosql/architecture/2015/09/13/notes-on-nosql-aerospike.html)  

<div class="note info">
  <h5>This post is a summary, so you have to read all references</h5>
  <p>
    This post is a brief summary and not intended to explain all related terms, but provides references instead. Please read all related materials. If some references missed - please improve the article.
  </p>
</div>


* TOC
{:toc}

# Difference between Relational and NoSQL databases

## Why NoSQL was reinvented

[NoSQL databases](https://en.wikipedia.org/wiki/NoSQL) ("non-SQL" or "non-relational") are not new, they have existed since the late 1960s and was generously forgotten in favor of relational databases, which was able to deal with data size and load and provide **strong consistency**, **ACID guarantees**, **transactions** and **rich query language**.

These key characteristics of **relational databases** was a direct result of **centralized architecture** and **relational databases** was perfect as [System of Record](https://en.wikipedia.org/wiki/System_of_record). The reverse side of this architectural approach, absence of **horizontal scalability**, wasn't critical, 'cause data size and load was relatively small and **vertical scalability** was enough to deal with.

Things changed with [Big Data](https://en.wikipedia.org/wiki/Big_data) era, which added new challenges and these challenges was exactly in area of the biggest weakness of relational databases. While requirements of a [System of Record](https://en.wikipedia.org/wiki/System_of_record) is widely known, let’s highlight key challenges of [Big Data](https://en.wikipedia.org/wiki/Big_data) area:

* **Velocity** - query/updates are exceptionally fast or large

* **Volume** - store a massive amount of data at rest

* **Variety** - The structure of records varies dramatically

**Centralized architecture** of **relational databases** become serious restriction to deal with BigData challenges.

## NoSQL databases as another trade-off for new area

*In partitioned databases, trading some consistency for availability can lead to dramatic improvements in scalability.*  
*--- Dan Pritchett, eBay*
{:.align_right}

The key to deal with BigData challenges was **horizontal scalability**. Current approach to achieve **horizontal scalability** is a **partitioned** and **distributed** system. 

But, as you can remember from the explained above, "*key characteristics of **relational databases** was a direct result of **centralized architecture***". So, **distributed** systems become **horizontally scalable**, but mostly lost (see below) benefits of **centralized architecture** - **strong consistency**, **ACID guarantees**, **transactions** and **rich query language**.

So, **relational** and **NoSQL** databases it’s a different solutions in a **trade-off** between **ACID guarantees**, **durability**, **integrity (transactional and referential)** and **strong consistency** on one side and **low latency**, **horizontal scalability**, **throughput** and **high availability** on the other side. 

As we can see, **NoSQL can't be comprehensive replacement for relational databases**, NoSQL it's just another **trade-off** to deal with new BigData challenges, but to be able to deal with 3V of BigData, some functionality of **relational databases** was sacrified and some functionality was added instead.

![Relational and NoSQL DB challenges](/assets/posts/2015-09-10-notes-on-nosql/image_00.png)

In general, RDBMS and NoSQL databases have different areas of applicability, where their strengths are fully used and impact from weakness is minimal: [System of Record](https://en.wikipedia.org/wiki/System_of_record) and [Big Data](https://en.wikipedia.org/wiki/Big_data). Pros and Cons of RDBMS and NoSQL systems it’s a direct match to their area of applicability.

Of course, different architectural solutions and tunable settings of Relational (RDBMS) and NoSQL databases may provide various balance between these corner cases, but this is always kind of trade-off.

## Why NoSQL become so popular

Well, the main reason for NoSQL popularity it's a current challenges. More and more usecases require **3V**, **high availability** and **continuous availability** and ready to sacrifice strong consistency, transactions and relational model. So "new" architectural approach of NoSQL is more suitable.

Another reason is the fact, that modern NoSQL databases, as was said above, become highly tunable and may provide various balance in the trade-offs described, like tunable and conditional consistency, persistency etc. vs scalability and performance. So weakness and performance of NoSQL become highly tunable in a big range.

The same is true for relational databases, so this is why "Space of requirements and limitations between corner cases" on the previous picture is so big.

So, there is no "silver bullet" or universal solution for every case. Every tool and configuration of a tool need to be chosen for particular case accordingly to project’s requirements. So question "What’s better, Relational or NoSQL database?" is meaningless until will be followed by “... for this particular case”.

## Most important trade-offs between relational and NoSQL databases

Here is the list of key differences between relational and NoSQL databases. Keep in mind, that modern databases are highly tunable, so some trade-offs may be changed by proper configuration:

1. **Centralized** and **Distributed** architecture as a key to strengths of both solutions (**Consistency**, **Transactions**, **ACID**, **Durability** and **Velocity**, **Volume**, **Availability**)
1. Mostly **Vertical** vs **Vertical** and **Horizontal scalability** due to **centralized** and **partitioned/replicated** storage
1. True **transactions** vs mostly **record-level atomicity**
1. **Relational model** with **normalization** vs **de-normalized non-relational model**
1. Advanced **Query Language (QL)** to **relational model** and **indexes** support vs very basic query language, indexes and **filtration** (multiple queries, de-normalization, data nesting used)
1. QL-level analytic vs **EntryProcessors**, **User Defined Functions (UDF)** and **distributed processing**
1. Manual **partitioning** vs Automatic **sharding** and **load distribution** in the cluster
1. Pre-configured **replication** (mostly **asynchronous**) vs Automatic **rack-aware replication** (**synchronous** and **asynchronous**)
1. Pre-configured **capacity** and mostly manual **disaster recovery** vs **Dynamic scalability** and **continuous availability**

We will overview most of them below.

## Distributed and Centralized architecture

The key architectural difference between RDBMS and NoSQL databases is a **distributed** (NoSQL) and **centralized** approach (RDBMS). Most other characteristics (positive and negative) are result of the solution chosen.

**Distributed storage** (and **Sharding**) is a key architectural solution to 2 of 3 Big Data challenges: **Velocity** and **Volume**.

This is **not true** that **CAP theorem** prohibits to create a distributed database with ACID transaction capability, but **strong consistency** and **ACID guarantees** with **transactions** support will seriously affect performance of a distributed database and, literally, eliminate almost all benefits from distributed architecture. 

## Horizontal scalability

**Distributed** applications are more **scalable**, than **centralized**. So NoSQL database usually are **horizontally scalable** at **near-linear scale** for **data volume** and **throughput** (with **data partitioning**, **sharding** and **replication** used).

This make NoSQL databases good to deal with **high data volume** and **high data velocity**.

## Transactions

**Transactions** are much more expensive, complex in a distributed environment and may affect performance, so this is one more trade-off with performance. This is the reason why transactions are, in general, not supported in NoSQL (*Some NoSQL implementations may offer similar functionality. For example, Oracle's NoSQL offers transactional control over data written to one node and let choose consistency across multiple nodes. For perfect consistency it will wait for each write to reach all nodes*).

Some NoSQL databases (for example Intersystems Cache) may provide full transaction support but with significant impact to performance, as was explained above.

Nevertheless, NoSQL database provide atomicity on for a single record change.

## Relational and non-relational data model

RDBMS are based on **relational model** of data with **pre-defined schema**. This  is a key for **referential data consistency** and **normalization** in RDBMS. In RDBMS data are normalized and stored in a set of tables with relations and **referential integrity constraints**.

NoSQL is based on **record-centric** approach (document oriented, graph, key-value, wide column etc.) which is a good for data **partitioning**, **scalability** and fast read-write. As a result, in NoSQL data stored in de-normalized form in a few independent tables. So there is no links to other keys, no relations between records, no automatic reference constraint checks.

In NoSQL data model usually have no pre-defined schema, schema is **flexible**. Data types are supported. On one hand this is a easy way to deal with 3rd challenge of Big Data - **Data Variety** and schema change, on the other hand pre-defined schema may guarantee more data consistency.

## Indexes support

As well as transactions, indexes implementation and maintenance is more complex in distributed databases and on data models with flexible schema. This means that NoSQL indexes support usually less advanced than in RDBMS due to trade-off with performance.

In most NoSQL databases indexes are **local** (each node store indexes for local data) and distributed over the cluster, so cross-partition search request will be sent to multiple (all) nodes in parallel. Some NoSQL databases may provide **global indexes** (which cover all data partitions) with different limitations.

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

In this map from [Peter Bailis' HAT not CAP paper](http://db.cs.berkeley.edu/papers/vldb14-hats.pdf), **models shaded in red cannot be fully available** in a distributed system.

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_1.png)

Highly available models (yellow):

* Read Uncommitted (RU)

* Read Committed (RC)

* Monotonic Atomic View (MAV)

* Item Cut Isolation (I-CI)

* Predicate Cut Isolation (PCI)

* Writes Follow Reads (WFR)

* Monotonic Reads (MR)

* Monotonic Writes (MW)

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

Also see [Paxos protocols](https://en.wikipedia.org/wiki/Paxos_(computer_science)).

### Split-brain for AP-systems

If system designed to be AP-compatible (available for data change and operate while partitioned), then it will be inconsistent and may be divided onto few partitions. Such state is named **[split-brain](https://en.wikipedia.org/wiki/Split-brain_(computing))**. 

[The challenging case for designers of AP-systems is to mitigate a par­tition’s effects on consistency and availability](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed). The key idea is to manage partitions very explicitly, including not only detection, but also a specific [recovery (merge) process](http://cdn.infoq.com/statics_s1_20150819-0313/resource/articles/cap-twelve-years-later-how-the-rules-have-changed/en/resources/fig1%20small.jpg) and a plan for all of the invariants that might be violated during a partition.

## ACID and BASE

[ACID (Atomicity, Consistency, Isolation, Durability)](https://en.wikipedia.org/wiki/ACID) is a set of properties that guarantee that database transactions are processed reliably. In the context of databases, a single logical operation on the data is called a transaction.

Here is a summary fo how the **ACID** properties are interpreted by a relational DBMS:

* **Atomicity** requires that each transaction is executed in its entirety, or fail without any change being applied.

* **Consistency** requires that the database only passes from a valid state to the next one, without intermediate points.

* **Isolation** requires that if transactions are executed concurrently, the result is equivalent to their serial execution. A transaction cannot see the partial result of the application of another one.

* **Durability** means that the the result of a committed transaction is permanent, even if the database crashes immediately or in the event of a power loss.

So under CAP-theorem restriction, distributed system can provide **BASE** guarantees:

* **Basically Available**: system does guarantee the availability of the data as regards CAP-theorem; there will be a response to any request. But, that response could still be ‘failure’ to obtain the requested data or the data may be in an inconsistent or changing state.

* **Soft state**: state of the system could change over time, so even during times without input there may be changes going on due to ‘eventual consistency’

* **Eventual consistency**: The system will eventually become consistent once it stops receiving input. The data will propagate to everywhere it should sooner or later, but the system will continue to receive input and is not checking the consistency of every transaction before it moves onto the next one. 

Most RDBMS can provide ACID guarantees. NoSQL, as a distributed system, can provide BASE and ACID under some configurations and restrictions.

## Continuous data availability

"Continuous data availability" is a next step after “High Data availability”, where unplanned downtime, although not desired, is still expected. 

In NoSQL databases this become possible because of distributed architecture, share-nothing approach and replication (**rack-aware** and **cross-datacenter**)

## Persistence to disk

There is different approaches when to confirm data write, usually this is tunable and this is one more trade-off between **reliability** and **performance**. 

Most RDBMS confirm write when data a persisted to file storage (data files, commit logs etc.). 

Most NoSQL databases confirm write when changes are applied in memory plus (for some systems and settings) are distributed over few nodes. Writes to disk usually batched and asynchronous for better performance.

It’s a highly discussible question - synchronous replication over the cluster(s) vs disk persistency.

# Common knowledge about NoSQL

## NoSQL Database Types

### Document-oriented databases

Pair each key with a complex data structure known as a document. Documents can contain many different key-value pairs, or key-array pairs, or even nested documents. The notion of a schema is dynamic: each document can contain different fields. 

Document databases provide the ability to query on any field within a document. 

Some products provide indexing options to optimize queries, some of these products provide the ability to analyze data in place, without it needing to be replicated to dedicated analytics or search engines. 

Examples: **MongoDB** and **CouchBase**

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

Other systems like **CouchBase** **retain all conflicting values** and push the responsibility to resolving conflict back to the user. 

Another approach, followed by **Cassandra**, is simply to assume the **latest value is the correct one**. 

For these reasons, inserts tend to perform well in eventually consistent systems, but updates and deletes can involve trade-offs.

## APIs of NoSQL databases

Most of the NoSQL databases provides their own API and bindings to different languages.

Also some of them may provide **REST**, [Apache Thrift](https://thrift.apache.org/) and **Memcached** protocol.

**JPA** is available through **ORM** frameworks like **Hibernate** or **EclipseLink.**

## Summary of strengths of NoSQL

Strengths of NoSQL solutions is a result of the architecture approach and trade-offs:

* Near-Linear horizontal scalability for data volume and queries volume

* Compromise between consistency and latency, configurable in many solutions

* Easy High Availability

* Dynamic Schema (some solutions) or at least more flexible schema

* Auto-sharding

* Remote code execution on the node, which contain data (entry processors, map-reduce) (some solutions)

* Automatic replication (master slave, peer-to-peer, XDC)

* Aggregation Frameworks

* Flexible schema

* Continuous data availability

## Summary of weakness of NoSQL

Like strengths, weakness of NoSQL solution is a result of the architecture approach and trade-offs:

* Basic support of transactions

* Basic support of indexes

* Limited support or absence of Query Language

* Non-relational model with limited integrity check abilities

* CAP-theorem limitations

* Weak consistency for some settings and databases

* Asynchronous persistency to disk vs performance

## Key characteristic of NoSQL DB

Here is the short list of NoSQL database characteristics which need to be considered while choosing NoSQL database:

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

# Usecases of different NoSQL solutions

This list is highly discussible and reflects personal experience of the author:

- **Cassandra**, **CouchBase** and **Aerospike** good as **System of Records** and they are competitors in the same area.

- **MongoDB** is good for analytics and as document-oriented DB where HA is not critical

- **HBase** is good to work with Hadoop

- **Redis** is good for cache, where durability is not required

- **Coherence** is a strong consistency system, so IO and network glitches may affect latency and throughput.

# Additional Materials, required to read

[Strong consistency models](https://aphyr.com/posts/313-strong-consistency-models)

[ACID](https://en.wikipedia.org/wiki/ACID)

[BASE: An Acid Alternative](http://queue.acm.org/detail.cfm?id=1394128)

[Difference between Linearizability and Serializability](http://stackoverflow.com/questions/4179587/difference-between-linearizability-and-serializability)

[What is eventual consistency?](http://www.quora.com/What-is-eventual-consistency)

[Top 5 Considerations When Evaluating NoSQL Databases](https://s3.amazonaws.com/info-mongodb-com/10gen_Top_5_NoSQL_Considerations.pdf)

[CAP : You Can’t Sacrifice Partition Tolerance](http://codahale.com/you-cant-sacrifice-partition-tolerance/)

[CAP : Please stop calling databases CP or AP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)

[CAP : CAP Twelve Years Later: How the "Rules" Have Changed](http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed)

