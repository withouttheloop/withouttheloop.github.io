---
title: React Flux Etc
author: Liam McLennan
date: 2014-05-29 20:32
template: article.jade
---

I've been enjoying working with [React](http://pluralsight.com/training/courses/TableOfContents?courseName=react-fundamentals) for a while now. I like it's emphasis on stateless, composable components. 

Unlike Angular, React does not give the developer much help with the design of their application. To address this the React team recently starting talking about [Flux](http://facebook.github.io/react/blog/), the architecture they use at Facebook. Flux has many pieces but the significant idea is that events no longer bubble back up the component hierarchy, instead they go out to an event aggregator that changes the application state, causing the UI to update. Much is made of the single directional data flow, state -> UI -> events -> aggregator -> change state -> UI etc. I like this design and I'll add one important rule:

<blockquote>
Use the same state for all related application specific components. Some components will get more data than they need, but having a single state model simplifies changes. 
</blockquote>

The other common criticism of React is that is does not include batteries (again compared to Angular). This doesn't bother me much since I have strong opinions and have therefore created my own version of most components that I need. 

The first missing piece that a developer is likely to encouter is a client-side router. I have heard of many people using the Backbone router, which is nice because it supports both html5 style pushstate routing and the older hash fragment style routing. I've been using [Flatiron Director](https://github.com/flatiron/director) which is fine.

For server communication I have become comfortable with Backbone's resource-oriented style. Angular provides $resource for this purpose. I use [resourceclient](https://github.com/liammclennan/resourceclient) because it is better. It uses Q promises instead of the broken jQuery promises and integrates with another library to provide object validation and type coercion. Where most similar libraries return json data without methods resourceclient is smart enough to return actual instances of your client-side models. 

Integrated with resourceclient I use [dbc](https://github.com/liammclennan/dbc) to validate that objects match certain requirements and to create constructors that enure objects are valid. 

For example, I might create a constructor like so:

    var Person = dbc.makeConstructor({
        name: [{validator:'type',args:['string']}],
        age: [{validator:'type',args:['number']}]
    });
    Person.url = '/api/person/';
    var personGateway = resourceclient(Person);

    var firstPerson = new Person({
        name: 'Alice',
        age: 58
    });

    personGateway.create(firstPerson).then(function (data) {
        // data is the model returned from the server
    });

The next missing piece is support for forms and validation. For that I use a set of purpose-built modules that aren't reusable at the moment.  

If you have to do client-side / JavaScript work then React is the best of the bunch. Enjoy. 