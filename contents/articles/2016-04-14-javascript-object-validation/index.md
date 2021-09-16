---
title: JavaScript object validation with JSON Schema
author: Liam McLennan
date: 2016-04-14 20:32
template: article.jade
---

Recently I wrote about [Building a Web Application with Node and Typescript](http://withouttheloop.com/articles/2016-03-24-node-web-ts/). One of the advantages of having JavaScript (Typescript) on both the client and server is that we can share validation logic between the two. 

<img src="validate.jpg" alt="validating parking" style="width:236px;margin:auto;"/>

JavaScript Object Validation
=============================

There are many ways of validating JavaScript objects. 

The Hapi framework includes a library called [Joi](https://github.com/hapijs/joi) that validates objects this way:

```
var Joi = require('joi');

var schema = Joi.object().keys({
    name: Joi.string().alphanum().min(3).max(30).required(),
    age: Joi.number().integer().min(0)
}).with('name', 'age');

Joi.validate({ name: 'abc', age: 103 }, schema, function (err, value) { });  
```

Another popular choice is [Flatiron Revalidator](https://github.com/flatiron/revalidator), which validates objects like this:

```
var revalidator = require('revalidator');

console.dir(revalidator.validate({ name: 'abc', age: 103 }, {
    properties: {
        name: {
        type: 'string',
        required: true,
        minLength: 3,
        maxLength: 30
        },
        age: {
        type: 'integer',
        minimum: 0,
        required: true
        }
    }
}));
```

Years ago I even [wrote my own](http://withouttheloop.com/articles/2013-02-24-dbc/) [object validation library called dbc](https://github.com/liammclennan/dbc). It served me well and is still powering validation on many public web applications. I would not use dbc today as there are now better options available. Dbc has some novel features that the other libraries don't have, such as creating auto-validating objects:

```
var AjaxOptions = dbc.makeConstructor({
        type:       [
                        {validator:'type', args:['string']},
                        {validator:'oneOf', args:[['GET','POST','PUT','DELETE']]}
                    ],
        url:        [{validator:'type',args:['string']}],
        dataType:   [{validator:'type',args:['string?']}]
    });
```

Here, `AjaxOptions` is a new JavaScript 'class' that will automatically validate when an object is created from it. 

It also has a feature that enforces function argument and return type validation. The following code adds validation to a function and guarantees that its argument is a `string` and its return value is an `AjaxOptions`:

```
AjaxOptions.fromCorsRequest = dbc.wrap(function (corsRequestText) {
        return new AjaxOptions(new CorsRequest(corsRequestText).getJSON());
    }, {
        0: [{validator:'type', args:['string']}]
    }, [{validator:'isInstance',args:[AjaxOptions]}]);
```

Everyone has their own way of declaratively specifying what a valid object looks like. If only there was some kind of standard....

<img src="chello.jpg" alt="chello"/>

JSON Schema
=========

JSON Schema is a standard that defines, among other things, a declarative format for specifying object validation rules. JSON Schemas are themselves defined in JSON, leading to the delightful situation of having [a JSON Schema that defines the schema for all JSON schemas](http://json-schema.org/schema). JSON schema is well considered and standardised making it an excellent choice for JavaScript object validation. Now all we need is a library that can validate a JavaScript object against a JSON schema. 

Jsonschema
----------

[Jsonschema](https://github.com/tdegrunt/jsonschema) is a library that validates javascript options against JSON schemas. 

```
 var validate = require('jsonschema').validate;

  var schema = {
    "type": "object",
    "properties": {
      "name": {
        "type": "string",
        "minLength": 3,
        "maxLength": 30
      },
      "age": {"type": "integer", minimum:0}
    },
    "required": ["name","age"]
  };
  
  console.log(validate({ name: 'abc', age: 103 }, schema));  
```

With this we have a standard, reusable validation strategy that we can use in the browser or in node. 

We can make things a little neater with a bit of convention. Here's a way of validating objects that have a `schema` method (code is ES2015):

```
import {validate} from 'jsonschema';

class Person {
    constructor(name,age) {
        this.name = name;
        this.age = age;
    }
    schema() {
        return {
            "type": "object",
            "properties": {
            "name": {
            "type": "string","minLength": 3,"maxLength": 30
            },
            "age": {"type": "integer", minimum:0}
            },
            "required": ["name","age"]
        }
    }   
}

function validateObjectWithSchema(obj) {
    return validate(obj, obj.schema());
}

let p = new Person("abc",103);
console.log(validateObjectWithSchema(p));
```

Final Thoughts
==============

To benefit from validation we need to hook it into our application. Common places to do this are:

* form validation in the browser
* request validation on the server

Both of these are fairly simple. I will show some examples in future posts.