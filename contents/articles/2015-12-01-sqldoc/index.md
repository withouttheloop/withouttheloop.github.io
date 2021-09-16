---
title: SqlDoc
author: Liam McLennan
date: 2015-12-01 20:32
template: article.jade
---

<img src="doc.png" alt="Great scott!" align="left" style="width: 300px;margin-right: 20px;margin-bottom: 20px;" /> One year ago I built an application data storage library, called [PostgresDoc](https://github.com/liammclennan/SqlDoc). The intent was, and is, to support document db semantics on top of relational databases. I have since renamed the project to **SqlDoc** to acknowledge that it now works with Postgresql and Sql Server.

The strengths of SqlDoc compared to a dedicated document db are:

* works with your existing database server
* proper, multi-document transactions
* supports joins
* supports a hybrid document / relational model
* easy to project to flat views for reporting and ETL

The strengths of SqlDoc compared to a relational database are:

* simple programming model
* No object / relational mapping

New API
========

I heard that people like the `IDocumentSession` API from RavenDB so I've [added it to SqlDoc](https://github.com/liammclennan/SqlDoc/blob/master/TestsCs/DocumentSessionAPITests.cs). This will make life easier for people who want to do data access via a component provided from an IoC container.

Creating a session
--------------

    var _documentSession =
      new DocumentSession<Guid>(SqlConnection.From(connectionString));

Adding a document
-----------

```
var _aDocument = new PersonCs
{
    _id = Guid.NewGuid(),
    Name = "Docsesh",
    Age = 90,
    FavouriteThings = new[] { "Golf", "Statue of liberty" }
};
_documentSession.Store(_aDocument._id, _aDocument);
```

Updating a document
-------------

```
_aDocument.Age += 1;
_documentSession.Update(_aDocument._id, _aDocument);
```

Deleting a document
-------------

    _documentSession.Delete(_aDocument._id, _aDocument);

Persisting the unit of work
-------------

    _documentSession.SaveChanges();

Load by id
--------

    var fresh = _documentSession.Load<PersonCs>(id);

Load by query
-------

```
var result = _documentSession.Query<PersonCs>(
"select data from PersonCs where Data.value('(/FsPickler/value/instance/idkBackingField)[1]', 'uniqueidentifier') = @id",
new Dictionary<string, object> { { "id", _aDocument._id } });
```

Why you don't need to put IDocumentSession in an IoC container
====

Rather than making your application code take a dependency on IDocumentSession consider instead having your application code returns its results, and then do your data access. That way there is a neat separation between pure application code responsible for logic and calculation and external integration code that does integration with external things like file systems and databases.

Here is the traditional way:

```
public class MyApp
{
  public MyApp(IDocumentSession documentSession) {}

  public void DoThing() {
    documentSession.Store(new Thing());
  }
}
```

To test that we need to mock documentSession.

The alternative is to keep the persistence plumbing out of your application:

```
public class MyApp
{
  public static Operation DoThing() {
    var thing = new Thing();
    return Operation.Insert(thing.Id, thing);
  }
}

// consumer
UnitOfWork.Commit(connection, new Queue<Operation>(MyApp.DoThing()));
```

This separates the calculation and the logical result of adding a `new Thing()` from the mechanism of doing so. To test `DoThing()` just call it and inspect the result. 
