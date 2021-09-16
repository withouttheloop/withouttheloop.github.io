---
title: PostgresDoc Support For Sql Server
author: Liam McLennan
date: 2015-02-13 20:32
template: article.jade
---

> Using sql server as a document database, with transactions, joins, unit of work and other good stuff

[PostgresDoc](https://github.com/liammclennan/PostgresDoc) is now the most ironically named ORM! Also, it doesn't support objects, relations or mapping, but it does now support Sql Server - at least the recent versions.

Historically, PostgresDoc has been a data access library providing serialization/deserialization, transactions, and a unit of work on top of the Postgresql json data type and associated indexes. [Recently I got around to implementing the same thing on top of Sql Server's XML data type and its indexes](https://github.com/liammclennan/PostgresDoc/wiki/SQL-Server-Support).

Here is the Doco
-------

PostgresDoc now supports SQL Server. SQL Server does not have the required JSON support, so the SQL Server version serializes to xml instead. The serializer supports .net classes, records and discriminated unions.

Create your store like this:

```
let storeSql = SqlStore "your connection string"
```

and create tables like this:

```
CREATE TABLE [dbo].[card](
  [Id] [uniqueidentifier] NOT NULL,
  [Data] [xml] NOT NULL,
 CONSTRAINT [PK_card] PRIMARY KEY CLUSTERED
(
  [Id] ASC
)
```

Inserting, updating and deleting work the same as for Postresql. Querying requires the use of Sql Server's XML querying capabilities.

Querying
-------

```
[
        "sourceUrl", box url
        "userId", box userId
]
|> query<Deck> store "select [Data] from [deck]
where Data.value('(/Deck/userId)[1]', 'uniqueidentifier') = @userId
and Data.value('(/Deck/sourceUrl)[1]', 'nvarchar(512)') = @sourceUrl"
```

There are [many more options available for querying sql server xml](http://www.brokenwire.net/bw/Programming/125/querying-xml-fields-using-t-sql).

Indexing
------

[Sql server xml indexes traditionally index everything](https://www.simple-talk.com/sql/database-administration/getting-started-with-xml-indexes/), which is convenient, bulky and slow.

[Since 2012 Sql Server also has selective indexing](https://www.simple-talk.com/sql/learn-sql-server/precision-indexing--basics-of-selective-xml-indexes-in-sql-server-2012/).
