---
title: Converting to TypeScript
author: Liam McLennan
date: 2014-09-25 20:32
template: article.jade
---

My most recent side project is a web based curl - [wurl](http://wurl.io) - intended to be [a convenient interface for experimenting with HTTP APIs](/articles/2014-02-09-aristotle/). 

Wurl is a useful tool that I created to satisfy a need - but it is also a project that I use to experiment with development techniques and technologies. 

In the browser we are limited to JavaScript, a dynamic language not easily integrated with whatever server-side technology is in use (in this case Asp.Net MVC and F#). To help prevent runtime JavaScript bugs I recently converted wurl from JavaScript to [TypeScript](http://www.typescriptlang.org/), a statically typed superset of JavaScript. The purpose of this post is to document that experience.

Where to Start?
-------------

It is not entirely obvious where to start a JavaScript -> TypeScript migration. I found that the best way is to start with bits of JavaScript that are small and have few dependencies. 

Converting a JavaScript Module to TypeScript
-------------------

Wurl has a JavaScript module used to contain its models. The `Template` model represents an HTTP request and how it is transformed. 

    ﻿define('models', [], function() {
    
        var Template = dbc.makeConstructor({
            id: [{validator:'type',args:['string']}],
            label: [{validator:'type',args:['string']}],
            request: [{validator:'type',args:['string']}],
            transformer: [{validator:'type',args:['string']}],
            view: [{validator:'type',args:['string']}]
        });
        Template.url = '/template';

        return {
            Template: Template
        };
    });

Step 1 was to rename the file with a .ts extension. Visual Studio immediately displayed a compilation error because the type of the `define` function is not known. `define` is a function that comes from the [smodules](https://github.com/liammclennan/smodules) (small modules) JavaScript module library. To be able to use smodules, a JavaScript library, from TypeScript I need a [type definitions file](http://www.typescriptlang.org/Handbook#writing-dts-files) that defines all the types exposed by smodules. To create a type definition for smodules I create a new file with the same name as the smodules library file, but with the extension .d.ts. In that file I add a type declaration for the define function. 

    ﻿declare function define(name:string, dependencies: string[], moduleFactory: any): void;

This tells TypeScript that define is a function, with arguments of type string, string array and any. Define does not return anything. I also add a declaration for the other function exposed by smodules:

    declare function require(name: string) : any;

The issue now is that TypeScript does not know where to look for the type definitions, so I add a reference to the models module:

    ﻿/// <reference path="packages\02-smodules.d.ts" />

Now when I try to compile I get no errors about smodules, instead the compiler complains about the call to `dbc.makeConstructor`. [dbc](https://github.com/liammclennan/dbc) is a library providing runtime type assertions. Here it is being used to define a runtime 'type' check. This compilation error is fixed according to the same strategy: create a .d.ts file for dbc containing:

    ﻿declare module dbc {
        function makeConstructor(spec: any): any;

        function check(o: any, spec: any, message?: string): void;
    } 

Using Well-Known JavaScript libraries
------------------------------

wurl makes use of many JavaScript libraries including:

    * underscore
    * react
    * jquery
    * q
    * toastr
    * codemirror

To be able to use these libraries from TypeScript I need type definitions for them all. Fortunately, there is a [github repository containing many type definition files](https://github.com/borisyankov/DefinitelyTyped) and they are available as nuget packages. 

Summary
-------

Wurl has perhaps 1,000 lines of JavaScript, so it is a small project. It took me about two hours to convert it to TypeScript, without having any idea what I was doing. It is easier to start a project with TypeScript from the beginning, however doing a JavaScript -> TypeScript conversion is perfectly feasible, even for larger projects.







