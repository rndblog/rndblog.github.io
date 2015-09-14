---
layout: post
title:  "Notes on NoSQL : CouchBase"
date:   2015-09-13 20:00:00
categories: NoSQL Architecture
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Principal Software Engineer and Grid Architect in San Francisco, USA. You can subscribe to my new posts in my <a href="http://sotnikdv.github.io">personal blog</a> or find me in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
editable: false
---

In this article we will overview one of the NoSQL databases, **CouchBase**. We will review architecture, key strengths and weakness and usage experience, if available.

This is the one of the few articles in "Notes on NoSQL" sequence  
[Notes on NoSQL : Basics](/nosql/architecture/2015/09/12/notes-on-nosql-basics.html)  
[Notes on NoSQL : MongoDB](/nosql/architecture/2015/09/13/notes-on-nosql-mongodb.html)  
[Notes on NoSQL : Cassandra](/nosql/architecture/2015/09/13/notes-on-nosql-cassandra.html)  
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

**CouchBase** is a wide-column database written on C++, so no GC, no heap and low latency with a lot of data, forked from CouchDB (??)

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

# Additional Materials, required to read

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

[MongoDB FAQ](https://www.mongodb.com/faq)

[MongoDB Multi-Data Center Deployments](http://s3.amazonaws.com/info-mongodb-com/MongoDB_Multi_Data_Center.pdf)

[Cassandra background-and-architecture](http://www.slideshare.net/yellow7/cassandra-backgroundandarchitecture)

[GridDynamics Cooking Cassandra](http://www.slideshare.net/Open-IT/cassandra-meetup-april-2014) presentation

[Using Apache Cassandra: What is this thing, and how do I use it?](http://www.slideshare.net/jeremiahdjordan/cassandra-intro-27171544)

[NoSQL Essentials: Cassandra](http://www.slideshare.net/frodriguezolivera/nosql-essentials-cassandra-15625311)
