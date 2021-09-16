---
title: Resourceserver as a backend for SPA and Backbone apps
author: Liam McLennan
date: 2013-07-15 20:32
template: article.jade
---

Some [reddit pundits were confused about resourceserver](http://www.reddit.com/r/javascript/comments/1i9rc5/http_resource_server/) and what the point of it might be. In this post I demonstrate the use of resourceserver as a backend for the todomvc SPA demo.

Using Resourceserver as a backend for an SPA / Backbone app
----------------------------

### Install [redis](http://redis.io/)

You're on your own for this one. It is not hard and can be accomplished on *nix or windows.

### Get resourceserver

```
git clone git@github.com:liammclennan/resourceserver.git
```

### Get Todomvc 

Todomvc is a reference app used to compare client-side web app libraries. It includes the same app (a todo app) built with a wide variety of libraries. We will use it as a client-side app to connect to resource server.

```
git clone git@github.com:tastejs/todomvc.git
```

### Enable redis

This is optional. By default resourceserver will just store resources in memory. Edit resourceserver's config.coffee:

```
module.exports =
 
   use_redis: true
 
   redis_config:
     host: 'localhost'
     port: 6379
```

Make sure the `redis_config` settings match your redis installation.

### Start resourceserver

In the directory where you cloned resourceserver:

```
npm start
```

### Modify todomvc 

We need to make a small modification to todomvc to enable server persistence. 

Modify architecture-examples/backbone/js/collections/todos.js:

```
-               localStorage: new Backbone.LocalStorage('todos-backbone'),
+               url: 'http://localhost:3002/todos',

```

### Enjoy

<img src="/articles/2013-07-15-resourceserverbackbone/demo.png" alt="Backbone and resourceserver"/>

