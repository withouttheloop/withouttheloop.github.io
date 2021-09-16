---
title: PostgresDoc
author: Liam McLennan
date: 2014-04-22 20:32
template: article.jade
---

Story so far
------------

Lately I have been moving towards more functional style programming. One way that you can categorize functional languages is those that have their own complete ecosystem (haskell, erlang, oCaml) and those that piggyback off an existing ecosystem, runtime and standard library (F#, scala, clojure). It is clear that the second category has a substantial advantage for adoption. The runtimes are well established, reliable and fast. The standard libraries and package managers provide the best guarantee possible that I will not be left missing a critical component. There are downsides too. The integration between the functional programming syntax and underlying standard library are well designed but awkward. However, on balance the piggyback functional languages (F#, scala and clojure) provide a practical and valuable way to move to functional programming. 

For me F# has some distinct advantages. I think types are one of the most important tools we have for writing working software, so dynamic languages like clojure have less appeal. Having spent 10 years working with .net the CLR is more approachable to me than the JVM. Finally, scala displeases me aesthetically. 

Recently, [my employer](http://readify.net/) sponsored a professional development activity where [I investigated the feasibility of applying functional programming to .net consulting projects](http://withouttheloop.com/articles/2014-01-14-fsharp-for-consulting-projects/). I found that it is practical, even simple, to partially or completely utilize functional programming via F# in this context. 

Struggle
--------

The last remaining awkward area was data access. I could not find a nice solution for data access from F#. Entity framework can be done, but involves a lot of compromise, essentially writing F# code in a C# style (the worst of both worlds). RavenDB seems like a good fit, which is why [many](http://ravendb.net/docs/1.0/client-api/fsharp) [have tried hard](http://colinbul.wordpress.com/2013/04/29/f-3-and-raven-db/) to make it work, but without support from the developers of RavenDB the result is disappointing. MongoDB may be web scale, but there [effort at supporting F#](http://blog.mongodb.org/post/59584347005/enhancing-the-f-developer-experience-with-mongodb) never made it out of alpha. 

What I wanted was a document database, with transactions, decent performance, simple querying and powerful indexes. I could not find one. 

Resolution
----------

[Postgresql](http://www.postgresql.org/) is an [amazing](http://www.wekeroad.com/2012/07/19/postgresql-rising/) open-source relational database. It ain't pretty but it is powerful. The latest version (9.3) includes [comprehensive support for working with structures serialized as json](http://www.postgresql.org/). It is possible to project queries over json documents, to query based on values in json documents and to create indexes (including unique indexes) on values in json documents. Being a typical relational database transactions are obviously available. Postgresql has all the basic features I want in a document database. All it needs is a nice, F# friendly, API.

PostgresDoc
-----------

For a long weekend project I built [PostgresDoc, a new API on top of postgresql](https://github.com/liammclennan/PostgresDoc) and [npgsql](http://npgsql.projects.pgfoundry.org/). It simplifies the syntax, optimizes for document data and adds an explicit unit of work pattern. Here is the examples from the readme:

Unit of Work API
----------------

    type Person = 
        { _id: System.Guid; age: int; name: string }
        interface IDocument with
            member x.tableName() = "People"
            member x.id() = x._id

    let store = { connString = "Server=127.0.0.1;Port=5432;User Id=postgres;Password=*****;Database=testo;" }

    let julio = { _id = System.Guid.NewGuid(); age = 30; name = "Julio" }
    let timmy = { _id = System.Guid.NewGuid(); age = 3; name = "Timmy" }
    
    // list is backwards so operations can be consed
    let uow = [ 
        Delete timmy
        Update { julio with age = 31 };
        Insert julio;
        Insert timmy;
        ]
    commit store uow

Document API
------------

### Query

    let peopleWhoAreThirty = 
        [ "age", box (30) ] 
        |> Map.ofList
        |> query<Person> store "select data from people where data->>'age' = :age"

The idea is that the unit of work is built up during execution by appending data operations (insert, update or delete). The example unit of work is really shorthand for:

    let uow = Delete timmy :: Update { julio with age = 31 } :: InsertJulio :: [Insert timmy]

The unit of work is committed in a transaction (beat that mongodb). 

Summary
-------

PostgresDoc is the document database that I wished I had. It provides a nice API for F# and seems to work!  