---
author: jsr
comments: true
date: 2011-06-08 13:53:14
layout: post
slug: schemas-for-schemaless-database
title: Schemas for schemaless databases
wordpress_id: 28
categories:
- MongoDB
---

Most of the modern NoSQL databases have eschewed the traditional RDBMS schema for a schemaless design. Databases like MongoDB, CouchDB, HBase, and Riak all allow you to store arbitrary new fields in your database without having to change any configuration. 

With this comes some great advantages. Development cycles and data management just go more quickly because there's less code to change (I can just update my Java or Python code without reconfiguring my Database). 

But there are still some challenges that a schema would make easier:  




 
  * Validation is hard without a schema. We're left to language specific validation rules that don't port to other environments

 
  * Multi-language development is difficult because we need to synchronize our data model across languages without any neutral format

 
  * There's no standard language neutral way to document data models

 
  * Many derivative software such as batch processing or aggregation engines would be easier with some formal type definitions






## Validation


Today most NoSQL stores leave validation as an exercise for the reader. This means that in your application code, you need to write lots of defensive code & logic to make sure data is valid before you put it into the store. 

Once it's there, it's very difficult to figure out if the data in your database is actually valid. Human errors, faulty software, or any number of defects or software upgrades could result in invalid data. 

Validation is difficult, especially with document-oriented data stores. Traditional SQL schemas don't really fit the bill for a few reasons:




  * Data is not stored in references, so a referential schema is more or less useless


  * SQL schemas are typically not "round-trip" compatible. In other words, I cannot generate a schema from my code, and then generate code from my schema


  * It's difficult to retain the highly dynamic nature of document oriented stores in conjunction with a strict schema



A great validation engine for document oriented database would survive these challenges. 



## Multi-language development


If you're writing clients to your data in multiple languages, you need to essentially recreate your schema in each language accessing your database. Modern tools like [Avro](http://avro.apache.org/), [Protobuffs](http://code.google.com/p/protobuf/), and [Thrift](http://thrift.apache.org/), and even older tools like [CORBA IDL](http://en.wikipedia.org/wiki/Interface_definition_language), [ASN.1](http://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One), and [DCE/RPC](http://en.wikipedia.org/wiki/DCE/RPC) provide smart workflows for dealing with multi-language environments because you can build a generic description of your data and generate language specific stubs for any environment you want to access it. 

It would be great to have this ability for document oriented stores



## Language neutral


Ideally a schema definition language would be external to the language being used to access the database. The schema should be the same regardless of which language I'm using to access the DB and let us work in whatever language is necessary for the job at hand



## Type systems for add on tools


When building things like Map Reduce jobs or processing pipelines, it's useful to be able to reason about the types of objects passing between phases in my pipeline. Jobs are significantly simpler if I can have some guarantees that, for example, each document contains specific fields so the system can validate objects before entering the pipeline. 



## Thoughts and next steps


I've been talking to [Brendan McAdams](http://twitter.com/rit) about a system that would bridge some of these gaps, specifically for mongoDB. Look for some follow up posts where we expand on our thoughts here

