---
layout: post
title: "DDIA Study -  Chapter 2 : Data Models and Query Languages (ENG)"
categories: DDIA
author: "Yongchan Hong"
---


# Chapter 2 : Data Models and Query Languages

> In this DDIA category, I will post a summary of Designing Data Intensive Applications (DDIA) from chapter 1 to 12.

In this chapter, we learn about various data models and query languages.

## Relational Model

A relational model is based on the relation of data and each relation is a collection of tuple.  
This model is a traditional model which requires explict **schema-on-write**. **Schema-on-write** will enforce all data to follow instructed schema. Thus, it may require migration if there is a new/altered field in data, which may cause downtime due to *ALTER TABLE* statement that copies whole table. Update will also take some time so users may put default value of NULL instead.   
This model may have **impedance mismatch** without ORM framework where there is a split between application code and database model. **One-to-many** relationship is an example for this situation since it may have different format of data in application and database.  
Relational Model can well serve **many-to-one** relationship or **many-to-many** relationship thanks to various joins with foreign key.  
> Hierarchial Model, Network Model and Relational Model
> 
> Hierarchial Model is a data model of IMS, the most popular database in 1970s. Hierarchial Model looks similar to JSON model but it had problem with **many-to-many** relationship since it did not support join at all.
> 
> Network Model is an unpopular solution for Hierarchial Model by CODASYL. This generalize hierarchial model by providing a pointer-like access path. Thus, in order to serach a record, application code should track all paths. Since it is complex for updating and querying, it is forgotten now.
> 
> Relational Model is a popular solution for Hierarchial Model. Relational Model simply lay out data in open. Relational database has a query optimizer which decides the order of execution in parts of the query and chooses which index to use. These choices will be an "access path". Unlike `join` is performed during `insert time` in CODASYL, SQL perform `join` in  `query time`. Relational Model was selected since it is much easier to query and update.  

## Document Model

Document Model is part of NoSQL where data are saved as a XML or JSON document form.  
This model is especially helpful when there are high write throughput and less restriction in schema (due to implicit **schema-on-read**). **Schema-on-read** will read schema when reading data. Thus, schema does not have to be equal for all documents.    
This model reduces **impedance mismatch** since it can perfectly express **one-to-many** relationship by XML/JSON format.  
Document Model may sometimes have better performance due to **storage locality**. When there is a frequent need to access an entire document, document model can simply access document compared to Relation Model.  
However, this model is not totally suitable for **many-to-one** relationship or **many-to-many** relationship since support for joins are weak. (It is similar to Hierachial Model but document model provides document reference).  


## Relational VS Document
Relational Model provides join and **many-to-many** relationship, while Document Model provides **schema flexibility**, better performance due to locality, and prevention of **impedance mismatch**.
Users should choose an appropriate model based on their situation.  
Recently, Relational Model and Document Model are converging. Most of the Relational Model provides querying/indexing for XML/JSON documents. Document Model provides relational-like join and automatic resolution of database reference (RethinkDB, MongoDB).  

## Query Language

### Imperative Query Language 
Imperative Language is used by most programming languages, which orders computer to perform  certain set of operations in a specific order. In query language, IMS and CODASYL are well known Imperative Query Language.

### Declartive Query Language 
Unlike Imperative Query Language, Declartive Query Language just requires user to specify the pattern of data (SELECT, WHERE etc). Query optimizer will then handle the order. SQL is well-known Declartive Query Language. Declartive Query Language is more concise to users as they do not have to know optimizing process. Also, Declartive Query Language is easily parallelizable compared to Imperative Query Language since it does not require specific instructions that may cause complexity.  
CSS selector also use Declartive Language which makes selection way easier than Imperative Language (Javascript DOM API).  

### MapReduce Query Language 
MapReduce is used to process big data, mostly used by Hadoop and MongoDB(limited). MapReduce is a mix of Imperative and Declarive Language. MapReduce has two components - map and reduce. Map basically searches data with matching query and emits key-value pair. Reduce aggregates a returned value from map and returns the value. Map and Reduce should be a pure function which cannot perform any additional queries and generate side effects.

## Graph Model
Graph Model is a suitable model for **many-to-many** relationships. A graph consists of two kinds of objects: vertices and edges. Usually, verticies mean people, events, locations, etc and edges mean a relationship between verticies. There are mainly two kinds of graph model - **Property Graph** model (Neo4j, Titan, InfiniteGraph) and **Triple-Stores** model (Datomic, AllegroGraph).  

### Property Graph
Property Graph model is a common graph model where verticies and edges consist:  
> Verticies:  
> - A unique identifier
> - A set of incoming edges
> - A set of outcoming edges
> - A collection of properties (key-value pairs)
> 
> Edges:
> - A unique identifier
> - The vertex at which the edge starts (the tail vertex)
> - The vertex at which the edge ends (the head vertex)
> - A label to desribe the relationship between two verticies
> - A collection of properties (key-value pairs)

Property Graph uses the Cypher query language.  
Vertices should be expressed like:  
>  (NAmerica:Location {name:'North America', type:'continent'})  

while edges should be expressed like:

>  (Idaho) -[:WITHIN]-> (USA) -[:WITHIN]-> (NAmerica)

### Triple-Stores
Triple-Stores model handles information in a three-part statement format: **subject**, **predicate**, **object**. For triple like (Chan, loves, Curry), Chan is the subject, loves is the predicate and Curry is the object. Object can be either a primitive datatype like string, or another vertex in the graph. Triple-Stores will have a shape like:  

> @prefix : <urn:example:>.  
> _:chan a :Person.  
> _:chan :name "Chan".  
> _:chan :bornIn _:seoul.  

Triple-Stores model can be closely linked to a semantic web and Resource Description Framework (RDF).  
SPARQL is a query language for Triple-Stores model which looks like: 

> ?person :bornIn / :within* ?location. #SPARQL

which is way simpler than Cypher language:

> (person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (location) # Cypher
