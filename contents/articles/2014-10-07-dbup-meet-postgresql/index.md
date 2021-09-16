---
title: DbUp (database migration) Support For Postgresql
author: Liam McLennan
date: 2014-10-07 20:32
template: article.jade
---

> DbUp is a .NET library that helps you to deploy changes to `SQL Server` databases. It tracks which SQL scripts have been run already, and runs the change scripts that are needed to get your database up to date.

DbUp is a simple, mature, sql script-based database deployment tool. I typically use it for all of my projects that use Sql Server. I like the idea so much that I have previously implemented similar systems for [CouchDb](https://github.com/liammclennan/couchpotato) and Postgresql. It occurred to me that instead of badly re-implementing half of the features of DbUp I should just add new database support to DbUp (open source right?).

> DbUp now supports Postgresql

Show me the Codez!
------------------

	var upgrader = DeployChanges.To
        .PostgresqlDatabase("Server=127.0.0.1;Database=testo;Port=5432;User Id=*****;Password=*******;")
        .WithScriptsEmbeddedInAssembly(Assembly.GetExecutingAssembly())
        .LogToConsole()
        .Build();

    var result = upgrader.PerformUpgrade();

> NOTE: at the time of writing this update has been merged but not yet made available via Nuget.