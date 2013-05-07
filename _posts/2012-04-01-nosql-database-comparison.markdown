---
author: jsr
comments: true
date: 2012-04-01 22:39:12
layout: post
slug: nosql-database-comparison
title: NoSQL Database Comparison
wordpress_id: 75
categories:
- Cassandra
- HBase
- MongoDB
- NoSQL
- Riak
---

People often ask to compare the various NoSQL solutions. There are a number of comparisons out there, in particular there's [Kristóf Kovács summary is pretty comprehensive](http://kkovacs.eu/cassandra-vs-mongodb-vs-couchdb-vs-redis). But I think that many comparisons focus on details that are secondary to the concerns of most developers. 

There's a lot of focus out there on mechanisms and implementation details, and not a lot of focus on the abstract guarantees that these databases can provide. I tend to break down the various NoSQL datastores according to the data models exposed to the developer and the invariants that the system can guarantee against those data models.

You can break these down into 4 characteristics of your data store: Data Model, Consistency, Availability, and Partitioning. In this post we'll look at the various design choices within each category and the implications of each choice. 

Full disclaimer, I work for 10gen, the makers of MongoDB, and before that I operated a MongoDB cluster in production for about a year. So I have a lot more in depth knowledge of MongoDB than the other datastores discussed in this document. Please point out any inaccurate statements in the content below and I'll happily correct it. 

We'll start with an overall picture of some of the more popular NoSQL stores and where they fit along these axes. We'll then dig into each axes and look at the choices and what it means for our application. 



## NoSQL Datastore Overview


<table>
    <tr>
        <th></th>
        <th>MongoDB</th>
        <th>Cassandra</th>
        <th>Riak</th>
        <th>HBase</th>
    </tr>
    <tr>
        <td>Data Model</td>
        <td>Documents</td>
        <td>Wide Column</td>
        <td>Key Value</td>
        <td>Wide Column</td>
    </tr>
    <tr>
        <td>Consistency</td>
        <td>Strong</td>
        <td>Eventual / Quorum</td>
        <td>Eventual / Quorum</td>
        <td>String</td>
    </tr>
    <tr>
        <td>Availability</td>
        <td>Single Master</td>
        <td>Multi Master</td>
        <td>Multi Master</td>
        <td>Single Master</td>
    </tr>
    <tr>
        <td>Partitioning</td>
        <td>Range or Hash</td>
        <td>Hash</td>
        <td>Hash</td>
        <td>Range</td>
    </tr>
</table>


## Data Model


The data model specifies how your application will format and store data in the database and how you can query and update that data later. Most of the NoSQL data stores fall into one of a few different data model options: 



### Key Value


Data is modelled as opaque blobs identified by a unique identifier. Key Value stores are the simplest data models available since there's relatively little work for the database to do. Your query language is limited to lookups by primary key, tho some vendors have extended their key value store with simple secondary indexes (eg. riak).

**Key Features**




  * High performance





### Wide Column


Data is modelled as a multi-level map comprising a Row Key, Column Family, and Column name. This model was popularized by [Google's BigTable](http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/en/us/archive/bigtable-osdi06.pdf). 

Queries must include the row key and can optionally include names of column families and columns to further restrict the query. 

This is a big improvement over the key-value API in terms of query flexibility as we can now filter on column names. However, we still lack the general ability to query on values as we can in traditional SQL queries. This model works pretty good for things like time series data where you need to store lots of values associate with a stream. But it can be challenging to model business data. 

**Key Features**




  * More flexible than key-value


  * Good at storing time series data





### Document


Data is modelled as hierarchical documents that consist of name value pairs nested within each other. This could be JSON, XML or any other similar syntax, tho most document oriented stores that are popular today work off of JSON. Documents benefit from being closer a closer abstraction to the objects we use in code. You tend to be able to take object trees and serializes them as JSON directly into the database, rather than requiring a mapping layer as is typical with relational or wide-column stores. 

**Key Features**




  * Maps closer to programming language


  * Easier to query, index





## Consistency


[Consistency](http://en.wikipedia.org/wiki/Consistency_(database_systems)) in the context of distributed databases refers to whether two clients trying to perform operations on objects in the database see the same or conflicting views of the world. The more they see the same view, the more "consistent" we say they are. Eric Brewer's work on the [CAP theorem](http://en.wikipedia.org/wiki/CAP_theorem) has guided much of the discussion around consistency today, pointing out that when a distributed database experiences failures, it must choose whether it wants to maintain Availability (e.g. clients can still perform operations on objects) or Consistency (e.g. clients see the same version of objects). We find that if we relax our consistency model, we can get stronger availability and vise versa. 



### Eventual Consistency


Eventual consistency is a relaxed consistency model typically employed in order to achieve higher availability in the database. In an eventually consistent system, writes to the database are _eventually_ visible to to all readers. This means that if I send a write request to the database and immediately try to read, then I may not read my own write. 

**Key Features**




  * High availability (as in, no downtime)


  * Might have inconsistent data






### Strong Consistency


Strong consistency ensures that if I perform a write operation to the database, other clients are guaranteed to see my write on the very next read. Databases that provide this level of guarantee often require small periods of un-availability during failures when they are temporarily unable to enforce this level of consistency. 

**Key Features**




  * Periods of unavailability while fail-overs happen


  * Clients can get stonger guarantees on object consistency






## Availability



Intimately tied to the Consistency model of your database is the Availability model. Most NoSQl stores use one of two approaches to achieve availability: single-master or multi-master. Note that pretty much any database can be highly available for eventually consistent reads at all times assuming part of the cluster is running. The really interesting part of availability is whether the system is available for writes, and whether it's available for consistent reads. 



### Single Master


In this type of data store, there is a single master that _owns_ each object in the database and multiple slaves that have eventually consistent copies of objects. This master is the only node that can process write requests or strongly consistent read requests for an object. If this master node fails or is otherwise unavailable, the system must go through a leader election process to find a new master. During this election process, the objects hosted at the master are unavailable for writes or consistent reads. Most systems still allow eventually consistent reads from slaves even during failures. 

**Key Features**




  * Enables strong consistency


  * Some unavailability during fail-overs






### Multi-Master


There are multiple masters for a single object and all of them are writable simultaneously. A client must write to a _quorem_ of available masters in order to write the object. This allows multiple nodes to be failed without losing the ability to write. However, during failures clients may fail writes if a quorum is not available, or you may have inconsistency if you write to less than a quorum or employ techniques like sloppy quorums. 

**Key Features**




  * Enables high availability


  * Some inconsistency expected






## Partitioning


Partitioning refers to how data is distributed across the cluster. 



### Range Based Partitioning


In Range Based Partitioning, contiguous ranges of values are stored together on nodes. This means that the system can easily support range query operations. However, it also means that load distribution may be focused on particular ranges of values. For example, if I have partitioned on a value with low cardinality, then I may have inefficient distribution of requests across partitions. However, it's pretty easy to implement hash partitioning on top of range partitioning by simply using a hashed value as your key. 

**Key Features**




  * Efficient range queries


  * Hash partitioning is easy to add






### Hash based partitioning


In Hash Based Partitioning, objects are distributed across the cluster according to a consistent hashing function. This enables efficient distribution of work for writes and reads by primary key, but it also means that a range query must hit every node on the cluster. 

**Key Features**




  * Good load distribution


  * Range queries are impossible or ineffiecient





## Summary


Choosing the right NoSQL store requires you understanding your use case and the tradeoffs of the platform you select. Most people move to NoSQL because some aspect of their relational datastore is inadequate. It's a useful exercise to think about what it is that you don't like about your existing database and what it is you want to get from NoSQL. I find that it's useful to think about this by asking the following questions: 





  1. What does my data look like and what kind of queries do I want to run? Are range queries important?


  2. When I have failures, would I rather have consistency violations, or unavailbility?



Compare the answers to these questions to the categories above and see where you fit.
