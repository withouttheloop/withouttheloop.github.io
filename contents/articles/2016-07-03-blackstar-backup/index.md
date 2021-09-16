---
title: Blackstar CMS Backup
author: Liam McLennan
date: 2016-07-03 20:32
template: article.jade
---

One of the most requested features for Blackstar CMS has been backup. Blackstar is an operational system so backup is important to maximise uptime and minimize data loss. 

Strategy
=======

Blackstar stores all its data in a Sqlite database specifically to facilitate a simple backup process. We can get a complete backup simply by backing up the database. 

Of course you can manually backup the database yourself, but Blackstar CMS will do it for you automatically. 

Blackstar's backup process is:

* on a schedule (every 10 minutes by default) the database is exclusively locked for the backup process
* if the databse has changed since the last backup then
    * copy the file to a backup location 
* release the lock so the application can continue running

It is necessary to lock the database so that the copy doesn't cause corruption. On a test database (1MB) the backup requires the database to be locked for 6ms, so the performance degredation is manageable. 

Implementation in Node.js
=================

The single threaded nature of node.js helps to make these sort of concurrent processes easy. When Blackstar starts it registers a callback on the backup frequency:

```javascript
setInterval(function () {
    // do backup
}, 10 * 60 * 1000 /* 10 minutes */);
```

To ensure a consistent backup the database needs to be locked for writes (reads are still OK). We can lock a sqlite database for writes using the `BEGIN IMMEDIATE` command. If an error occurs while the database is locked we still want to ensure the lock is released, so the process is wrapped in `try {} finally {}`.

```javascript
try {
    db.run("BEGIN IMMEDIATE");
    // do things with an exclusive write lock
} finally {
    db.run("COMMIT");
}
```

The whole thing is done synchronously, since we are blocking the server anyway. The file hash is calculated using node's `crypto` module, which delegates to OpenSSL.

```javascript
export function calculateHash(data:Buffer) : string {
    const hash = Crypto.createHash('sha256');
    hash.update(data);
    return hash.digest('hex');
}
```

If the hash of the current database is different since the last time the backup function ran then the database is copied to the backup destination.

Summary
======

The backup solution described above will be sufficient for most scenarios. For anything else it is easy enough to build your own backup process using a variation on the same technique (lock the database, take a copy, unlock the database). The [sqlite online backup API](https://www.sqlite.org/backup.html) provides some options. 