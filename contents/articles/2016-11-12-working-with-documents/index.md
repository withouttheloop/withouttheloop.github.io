---
title: Document Storage in Sql Server 2016 (JSON) and Postresql
author: Liam McLennan
date: 2016-11-12 20:32
template: article.jade
---

<style>
table {
    text-align: left;
    font-size: small;
    font-family: consolas;
    background-color: #eee;
    margin-bottom: 15px;
}
td {padding: 4px;}
</style>

Why?
====

Some problems map well to documents, or resources. When your data is a set of relatively independent collections then document storage will be much easier than relational. Anywhere that REST makes sense for resources documents make sense for data storage. Jeremy Miller [wrote a bit about why workring with documents can be efficient](https://jeremydmiller.com/2016/06/14/schema-management-with-marten-why-document-databases-rock/) in the context of Marten, the .net document database and event store library for Postresql. If you are not ready to embrace Marten it is possible to start working with documents, in Sql Server or Postresql, easily. 

SqlDoc
=====

[SqlDoc](https://github.com/liammclennan/SqlDoc) is sort of like Marten, but far less ambitious. It just does document data, in fact it is currently limited to these operations:

* run an arbitrary script 
* create a table to store documents
* insert a document
* update a document
* delete a document
* retrieve a document
* query for a documents using database indexes

It is a limited feature set but sufficient for a large class of applications. 

I am writing about SqlDoc again because I have recently added support for storing documents as JSON in Sql Server 2016, bringing the storage options to:

* postgresql 9.3 or greater
* Sql Server 2005 or greater (XML)
* Sql Server 2016 or greater (JSON)

Storing Document Data in Sql Server 2016 as JSON
===============

The first step is to create a table to store documents. SqlDoc expects a table per type stored. To store a type called `Person`:

```
CREATE TABLE [dbo].[Person](
	[Id] [uniqueidentifier] NOT NULL,
	[Data] [nvarchar](max) NOT NULL,
 CONSTRAINT [PK_person] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)
```

What the `Person` type is does not matter, because it will be serialized and stored as text in the `[Data]` column of the `Person` table. For this example let's use:

```
public class Person 
{
    public Guid _id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public string[] FavouriteThings { get; set; }
}
```

To work with our document database we create an `IDocumentSession` from a Sql Server connection string, like this:

```
var _documentSession =
            new DocumentSession<Guid>(SqlConnection.From(
                ConfigurationManager.AppSettings["ConnSqlXml"]))
```

`IDocumentSession` contains the obvious thing, and is defined as:

```
public interface IQuerySession<TKey>
{
    TVal Load<TVal>(TKey id) where TVal : class;
    IEnumerable<TVal> Query<TVal>(string sql, Dictionary<string, object> parameters = null);
}
public interface IDocumentSession<TKey> : IQuerySession<TKey>
{
    void Delete(TKey id, object datum);
    void SaveChanges();
    void Store<TVal>(TKey id, TVal entity) where TVal : class;
    void Update<TVal>(TKey id, TVal entity) where TVal : class;
}
```

`TKey` is the type of the field used as a primary key. `Guid` is a good choice. So with an `IDocumentSession` we can insert, update, delete, load and query documents. An insert is done like this:

```
var _aDocument = new PersonCs
{
    _id = Guid.NewGuid(),
    Name = "Docsesh",
    Age = 90,
    FavouriteThings = new[] { "Golf", "Statue of liberty" }
};
_documentSession.Store(_aDocument._id, _aDocument);
_documentSession.SaveChanges();
```

Of course we can enqueue many operations before calling `SaveChanges()` to persist the changes in a transaction. The `Person` table will now contain the following data:

| Id  | Data |
| ------------- | ------------- |
| 7BC6192F-1B78-4C31-9537-088EA4205871  | {"_id":"7bc6192f-1b78-4c31-9537-088ea4205871","Age":90,"Name":"Docsesh","FavouriteThings": ["Golf","Statue of liberty"]}  |

SqlDoc serialized the `Person` object to JSON, then inserted it into a table with the same name as the type (Person).

Updates are hopefully obvious:

```
_aDocument.Age += 1;
_documentSession.Update(_aDocument._id, _aDocument);
_documentSession.SaveChanges();
```

Now the database has:

| Id  | Data |
| ------------- | ------------- |
| 7BC6192F-1B78-4C31-9537-088EA4205871  | {"_id":"7bc6192f-1b78-4c31-9537-088ea4205871","Age":91,"Name":"Docsesh","FavouriteThings": ["Golf","Statue of liberty"]}  |

To delete a document:

```
_documentSession.Delete(_aDocument._id, _aDocument);
_documentSession.SaveChanges(); 
```

To load a document by its primary key:

```
var fresh = _documentSession.Load<Person>(_aDocument._id);
```

Note that this relies on the database primary key column being `Id`. 

Querying on Indexes
------

Querying without an index means a table scan, which is too slow for real use. [MSDN describes the recommended strategy for indexing JSON data](https://msdn.microsoft.com/en-us/library/mt612798.aspx), essential define a computed column for the value to be indexed and index the computed column. 

For our person example, we might wish to query by `Name`, so we start by adding a computed column:

```
ALTER TABLE person
ADD Name AS JSON_VALUE([Data], '$.name')  
``` 

The Sql Server table now appears as follows:

| Id  | Data | Name |
| ------------- | ------------- | ---------- |
| 7BC6192F-1B78-4C31-9537-088EA4205871  | {"_id":"7bc6192f-1b78-4c31-9537-088ea4205871","Age":91,"Name":"Docsesh","FavouriteThings": ["Golf","Statue of liberty"]}  | Docsesh |

And we have a `Name` column that can be indexed for querying according to the usual practice in Sql Server. 

```
CREATE NONCLUSTERED INDEX IX_Person_Name
ON Person ([Name])  
```

Finally, we can query our index to search by name:

```
var result = _documentSession.Query<PersonCs>(
    "SELECT [Data] from person where [Name] = @name",
    new Dictionary<string, object> { { "name", _aDocument.Name } });
```

Alternative API
------------------

Most people seem to like the `IDocumentSession` API (inspired by Marten and RavenDB) for working with document storage. There is an alternative API based on accumulating operations in a queue, and then commiting those operations.

```
var connection = SqlConnection.From(ConfigurationManager.AppSettings["ConnSqlXml"]);
var _uow = new Queue<Operation<Guid>>();
_uow.Enqueue(Operation.Insert(_aDocument._id, _aDocument));
UnitOfWork.Commit(connection, _uow);

var resultst = QueryFor<Person>.For(
    connection,
    "SELECT [Data] from person where [Name] = @name",
    new Dictionary<string, object> { { "name", _aDocument.Name } });
```
 
Both APIs accomplish the same goal. The difference may come down to how much you like dependency injection. 