---
title: Postgresql as a Nosql Document Store 
author: Liam McLennan
date: 2014-09-30 20:32
template: article.jade
---

[Postgresql](http://www.postgresql.org/) makes a [fairly decent schemaless Nosql document database](http://www.infoq.com/news/2014/05/postgresql-9-4). It is the only database I can think of that:

* is mature
* is fast
* has an extensive plugin ecosystem
* has ACID compliance
* supports schemaless document storage
* supports relational data storage
* can join across documents and relational data
* can index into documents as well as relational data
* has built-in full text search

Postgres already had most of this. The only thing they had to add to facilitate document storage was a json data type and the ability to create indexes on json properties. 

An Example
----------

If I want to store Person documents I might create a table called `Person` with a json column to hold the document:

	create table Person ( data json NOT NULL );

The documents will have a unique identifier `_id` which I need to be able to query on, so I add an index:

	CREATE UNIQUE INDEX people_id ON Person ((data->>'_id'));

Then the query to select a document is:

	select data from Person where data->>'_id' = 'A158CCB9-BB68-4FC2-8123-79B32053B2A3'

Better Tooling
--------------

A typical application scenario will make changes to multiple documents. We can take advantage of Postgres's transaction support by accumulating changes and commiting them within the same transaction. If there are no problems then the changes are persisted, but if something goes wrong the entire transaction is reverted, leaving the database in a consistent state. 

I have been working on a project I call [PostgresDoc](https://github.com/liammclennan/PostgresDoc) to make working with postgres documents easier. Currently it is an F# project but I will add a C# wrapper soon. Use it like so

	// accumulate a stack of changes
	let julio = { _id = System.Guid.NewGuid(); age = 30; name = "Julio" }
	let uow = [Insert julio]
	let uow2 = Update { julio with age = 31} :: uow
	let uow3 = Delete julio :: uow2

	commit store uow3

This approach is great for testing (just make assertions against the unit of work) and is free from many of the awful consistency problems that plague ORMs such as NHibernate and Entity Framework. 

Queries are easy too:

	let peopleWhoAreThirty = 
    	[ "age", box (30) ] 
	    |> query<Person> store "select data from people where data->>'age' = :age"

More
------

* [Postgresql JSON Datatypes](http://www.postgresql.org/docs/9.4/static/datatype-json.html)
* [Postgresql JSON Functions](http://www.postgresql.org/docs/9.4/static/functions-json.html)
* [PostgreSQL 9.4 - Looking Up (With JSONB and Logical Decoding)](http://www.craigkerstiens.com/2014/03/24/Postgres-9.4-Looking-up/)