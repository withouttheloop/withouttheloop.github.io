---
title: Hosted backbone server
author: Liam McLennan
date: 2012-02-09 20:32
template: article.jade
---

This morning I published my [zero-config backbone.js server](https://gist.github.com/2956872). 

It is an in-memory, restful server that implements the protocol required by Backbone.js, which is basically the same as couchdb except that 'create' returns the new id. As such it can be used for any purpose requiring temporary resource persistence. The service provides no durability at all. If it restarts all of the data is lost.

I'm now running a public instance at http://withouttheloop.com:3002. It works on my machine, don't blame me if it crashes your space shuttle. No warranty of any kind. Obviously. 

To use with backbone set your model or collection's url property to `http://withouttheloop.com:3002/[yourresourcetype]`. Have a look at [this jsfiddle](http://jsfiddle.net/ynkJE/24/). If you run the example and look in Chrome's developer tools networking tab you will see:

### Request

    POST /books HTTP/1.1
    Host: withouttheloop.com:3002

### Response

    HTTP/1.1 200 OK
    X-Powered-By: Express

    {"title":"Midnight in the garden of good and evil","author":"John Berendt"}

The model is persisted to the hosted backbone server. 

The set of supported operations (which match those required by backbone.js) is:

Create
------

    $ curl -X POST http://withouttheloop.com:3002/people -H "Content-Type: application/json" -d '{"name": "Bob", "age":"28"}'

    => {id: 1}

Update
------

Bob had a birthday

    $ curl -X PUT http://withouttheloop.com:3002/people/1 -H "Content-Type: application/json" -d '{"name": "Bob", "age":"29", "id": 1}'

    => OK

Read the Collection
-------------------

    $ curl http://withouttheloop.com:3002/people -H "Content-Type: application/json"

    => [{"name":"Jane","age":"32","id":0},{"name":"Bob","age":"29", "id":1}]

Read a Resource
---------------

    $ curl http://withouttheloop.com:3002/people/1 -H "Content-Type: application/json"

    => {"name":"Bob","age":"29", "id":1}

Destroy
-------

    $ curl -X DELETE http://withouttheloop.com:3002/people/1 -H "Content-Type: application/json"

    => OK
