---
title: Backbone server
author: Liam McLennan
date: 2012-02-08 20:32
template: article.jade
---

[Backbone-Server](https://github.com/liammclennan/backbone-server) is a zero configuration backbone.js server. For teaching, experimentation and debugging it provides an instant server-side for backbone.js apps. Once installed it is started like so:

    node rasterver.js

Then from the client you can work with models:

    var Person = Backbone.Model.extend({}),
    People = Backbone.Collection.extend({
        model: Person,
        url: 'http://localhost:3000/People'
    }),
    people = new People();
    people.add({name: 'John', age: 25});
    people.at(0).save();

That is enough to get the model saved to the server. Fetch, update and destroy all work properly. 

The server stores the collection data in memory, so when the server stops the data is lost. This is not a good choice for production. 