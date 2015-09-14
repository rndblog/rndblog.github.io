---
layout: post
title:  "Notes on NoSQL : Redis"
date:   2015-09-13 18:00:00
categories: NoSQL Architecture
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Principal Software Engineer and Grid Architect in San Francisco, USA. You can subscribe to my new posts in my <a href="http://sotnikdv.github.io">personal blog</a> or find me in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
editable: false
---

**Redis** is the open-source (BSD license ) [key-value cache and store](http://redis.io/documentation), written in C. [Single-thread](http://redis.io/topics/clients) and [memory-optimized](http://redis.io/topics/memory-optimization). [Used by](http://redis.io/topics/whos-using-redis) Twitter, GitHub,  Pinterest, Craigslist, Digg, StackOverflow, Flickr.

This is the one of the few articles in "Notes on NoSQL" sequence  
[Notes on NoSQL : Basics](/nosql/architecture/2015/09/12/notes-on-nosql-basics.html)  
[Notes on NoSQL : MongoDB](/nosql/architecture/2015/09/13/notes-on-nosql-mongodb.html)  
[Notes on NoSQL : Cassandra](/nosql/architecture/2015/09/13/notes-on-nosql-cassandra.html)  
[Notes on NoSQL : HBase](/nosql/architecture/2015/09/13/notes-on-nosql-hbase.html)  
**[Notes on NoSQL : Redis](/nosql/architecture/2015/09/13/notes-on-nosql-redis.html)**  
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

[Value typed, base types supported](http://redis.io/topics/data-types-intro): [binary-safe strings, lists, sets, sorted sets, maps, bit arrays and HyperLolLogs](http://redis.io/topics/data-types) (probabilistic data structure used in order to count unique things, technically this is referred to estimating the cardinality of a set). 

Value typed until record exists. JSON is **not** supported.

[Keyspace notifications](http://redis.io/topics/notifications) via [Pub/Sub](http://redis.io/topics/pubsub) and Pub/Sub available.

# Architecture

There are three different architecture which is possible in Redis: single partition with replication (optional), Sentinel HA and Redis Cluster. 

Due to asynchronous replication and and persistence, Redis will not prevent data loss in any of those schemas, just will minimize it.

## Single Partition with Replication (optional)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_13.png)

### Replication

Replication is [based on asynchronous pre-configured master-slave replication](http://redis.io/topics/replication).

Replicas may serve read requests for scalability, but due to asynchronous replication data may be stale. Master is able to serve read-write. 

Starting 2.8 Redis sent only difference while replication.

Starting with Redis 2.8, it is possible to configure a Redis master to accept write queries only if at least N slaves are currently connected to the master. Detection is asynchronous.

**Also automatic restart of the master without persistency will lead to erasing of the replica’s storages.**

### Sharding

[Sharding (partitioning) is implemented in a 3 different ways on the different parts of a software stack](http://redis.io/topics/partitioning): client side partitioning, proxy assisted partitioning and query routing which will be overviewed in a Redis Cluster.

Some features of Redis will not be available with partitioning:

* Operations involving multiple keys are usually not supported. For instance you can't perform the intersection between two sets if they are stored in keys that are mapped to different Redis instances.

* Redis transactions involving multiple keys can not be used.

* The partitioning granularity is the key, so it is not possible to shard a dataset with a single huge key like a very big sorted set.

* When partitioning is used, data handling is more complex, for instance you have to handle multiple RDB / AOF files, and to make a backup of your data you need to aggregate the persistence files from multiple instances and hosts.

* Adding and removing capacity can be complex. For instance Redis Cluster supports mostly transparent rebalancing of data with the ability to add and remove nodes at runtime, but other systems like client side partitioning and proxies don't support this feature. However a technique called Pre-sharding helps in this regard.

#### Client side partitioning

Client side partitioning means that the clients directly select the right node where to write or read a given key. Many Redis clients implement client side partitioning.

[TODO add schema]

#### Proxy assisted partitioning

Proxy assisted partitioning means that our clients send requests to a proxy that is able to speak the Redis protocol, instead of sending requests directly to the right Redis instance. The proxy will make sure to forward our request to the right Redis instance accordingly to the configured partitioning schema, and will send the replies back to the client. The Redis and Memcached proxy [Twemproxy](https://github.com/twitter/twemproxy) implements proxy assisted partitioning.

[TODO add schema]

### Fault-tolerance

There is no High Availability in this case and automatic failover.

## Sentinel High Availability

[Redis Sentinel](http://redis.io/topics/sentinel) provides high availability abilities for Redis:

* automatic failover

* monitoring

* notifications

* configuration provide (service discovery for clients)

Sentinel is a distributed system of a Sentinel processes with auto-discovery.

The sum of Sentinels, Redis instances (masters and slaves) and clients connecting to Sentinel and Redis, are also a larger distributed system with specific properties.

[TODO add schema]

Quorum (configurable) of Sentinel processes may decide that master is dead and elect new master from slaves informing clients. Client **need to support Sentinel**.

### Replication

The same as for single master schema, but in case of failover slave may become a master. In this case part of the changes [will be lost](https://aphyr.com/posts/283-call-me-maybe-redis) due to **eventual consistency**. 

### Sharding 

The same as for single master schema.

### Fault Tolerance

Will survive fail of master, Sentinel process and slaves until there are enough replicas accessed and Sentinel quorum.

## Redis Cluster

Query routing means that you can send your query to a random instance, and the instance will make sure to forward your query to the right node. 

[Redis Cluster](http://redis.io/topics/cluster-tutorial) implements an hybrid form of query routing, with the help of the client (the request is not directly forwarded from a Redis instance to another, but the client gets redirected to the right node).

Redis Cluster is the preferred way to get **automatic sharding** and **high availability**. It is generally available and production-ready as of [April 1st, 2015](https://groups.google.com/d/msg/redis-db/dO0bFyD_THQ/Uoo2GjIx6qgJ)

![image alt text](/assets/posts/2015-09-10-notes-on-nosql/image_14.png)

[TODO Rewrite schema]

### Sharding

Redis cluster divides keyspace onto 16384 hash slots (CRC 16 mod 16384) and assign range to every node, so every node in a Redis Cluster is responsible of a subset of the hash slots. Move of the hash slot is automatic.

Redis Cluster supports multiple key operations as long as all the keys involved into a single command execution (or whole transaction, or Lua script execution) all belong to the same hash slot. The user can force multiple keys to be part of the same hash slot by using a concept called **hash tags**.

### Replication

In order to remain available when a subset of master nodes are failing or are not able to communicate with the majority of nodes, [Redis Cluster uses a master-slave model](http://redis.io/topics/cluster-spec) where every hash slot has from 1 (the master itself) to N replicas (N-1 additional slaves nodes)

Master and slave roles are pre-configured, also this is possible to configure replica to serve particular master.

As well as for Sentinel, slave may become a master. In this case part of the changes will be lost due to **eventual consistency**. 

In a Redis Cluster replica may be assigned to the master automatically and automatically migrated.

### Fault Tolerance

In case of master fail, slave will be elected as a new master. If master and slaves failed, Redis cluster will not be able to continue serving of the partition.

Split-brain is prevented by majority principle.

# Data consistency model

Until fail this will be a **strong (linear, due to single master and single thread) view consistency** for master and **eventual consistency** for replicas. In case of failover - **eventual consistency** with data loss.

# ACID guarantees

Durability is not guaranteed.

# CAP-theorem

CP for single master.

As for cluster this is depends on implementation. For Redis Cluster can be considered as AP until split discovery and CP after (transition with data loss).

# Split-brain and recovery from a split brain

Split brain is prevented by Sentinel and Redis Cluster by majority.

# Rack-aware replication and cross-datacenter replication

There is no rack-aware replication inside Redis, but [Redis Labs](https://en.wikipedia.org/wiki/Redis_Labs) declared [rack-awareness](https://redislabs.com/redis-enterprise-documentation/rack-zone-awareness) (as well as HA) in their product, [Redis Labs Enterprise Cluster](https://redislabs.com/redis-enterprise-documentation/overview), which is based on Redis.

# Dynamic scalability

Supported for replicas, Sentinel and Redis Cluster

# Persistence

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

# Query Language, Entry Processor, User Defined Functions and Bulk Writes

Query Language is not available, [system of commands instead](http://redis.io/commands), nevertheless only basic functionality for records (get/put, increment and queue operations) available.

- Is Query Language and bulk writes available

- Is Entry Processor and User Defined Functions supported

- Is Aggregation functionality available

Also available [User Defined Functions on LUA which can be evaluated on the node](http://redis.io/commands/eval), which is a good replacement for bulk operations in some cases.

[Mass upload](http://redis.io/topics/mass-insert) and [pipelining](http://redis.io/topics/pipelining) (bulk operations) is available. Also [transactions](http://redis.io/topics/transactions) (batch) may be used

# Indexes (local and global)

There is no indexes, Redis is a pure key-value storage with access by key only.

# Traffic encryption and access control

[Redis does not support encryption](http://redis.io/topics/encryption). In order to implement setups where trusted parties can access a Redis instance over the internet or other untrusted networks, an additional layer of protection should be implemented, such as an SSL proxy or [Slipped](http://www.tarsnap.com/spiped.html).

[Redis is designed to be accessed by trusted clients inside trusted environments](http://redis.io/topics/security). This means that usually it is not a good idea to expose the Redis instance directly to the internet.

In the same time Redis:

* can bind specific interface

* have tiny layer of authentication (password, pre-defined in **redis.conf** configured file)

* can be configured to disable or rename particular commands in **redis.conf**

# Caching schema

Redis is a pure in-memory storage with persistence and replication. Also can work as [LRU cache](http://redis.io/topics/lru-cache).

Records [may have expiration setting](http://redis.io/commands/expire).

# Business model and commercial support

[Community Support](http://redis.io/support).

Commercial Support available for [Managed Redis](https://redislabs.com/) instances from [Redis Labs](http://redislabs.com/) (the official sponsor of the Redis Project).

# Usage experience

Results are next:

* Extremely simple configuration and [very fast](http://redis.io/topics/latency-monitor), [hundreds thousands tps](http://redis.io/topics/benchmarks).

* Used as a cache for auth token management, for password bruteforce protection (counters in Redis)

* Redis Sentinel used for master-slave replication.

* No issues for the last year when used as a cache
