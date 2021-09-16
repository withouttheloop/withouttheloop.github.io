---
title: dbc 2.0.0
author: Liam McLennan
date: 2013-08-24 20:32
template: article.jade
---

Today I released a new version of my [JavaScript Design-by-Contract Library (dbc)](https://rawgithub.com/liammclennan/dbc/master/docs/dbc.html). As well as documentation and other minor improvements the two big new features are:

makeConstructor
=============

MakeConstructor generates a constructor from a spec. The constructor validates that the created objects meet the spec. ie

        var Person = dbc.MakeConstructor({
           name: [{validator:'type',args:'string'}],
           age: [{validator:'type',args:'number'}]
        });
        Person.prototype.canDrive = dbc.wrap(function () { 
             return this.age > 16;
        }, null, [{validator:'type',args:'number'}]);
        var p = new Person('John', 22);
        p.canDrive(); // true 

wrap
===

Wrap returns a wrapped function that applies 'specs' validators to the functions arguments and 'returnSpec' validators to the return value. eg

    var add = dbc.wrap(function (f,s) { 
        return f + s;
    }, {
        // validators for the first arg
        0: [{validator:'type', args:['number']}],
        // validators for the second arg
        1:[{validator:'type', args:['number']}]
    },
        // validators for the return value 
        [{validator: 'type', args:['number']}]);

Dbc is available on [github](https://github.com/liammclennan/dbc) or via npm 

    npm install dbc

