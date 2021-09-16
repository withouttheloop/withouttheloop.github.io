---
title: Handlebars Byte Order Mark!
author: Liam McLennan
date: 2012-08-01 15:00
template: article.jade
---

Byte order mark (BOM) is a unicode character that signals the byte order of a UTF text file. If displayed in a text editor it most often appears as `ï»¿`. 

Sometimes, when using [Handlebars.js](http://handlebarsjs.com/) to produce markup you will see extraneous whitespace inserted into the DOM immediately prior to your rendered template. In the chrome developer tool, this extra whitespace appears as `" "`. In firebug it appears as EF BB BF.

This problem seems to occur when using the [handlebars.js precompilation feature](http://handlebarsjs.com/precompilation.html) to precompile templates on the server and combine them into a single script. The precompiler doesn't remove the BOM marks when compiling the templates so they end up in the DOM, messing with your layout. 

> The solution is to make sure that the template files do not include BOMs. You can use Notepad++, Sublime Text or any good text editor to save a file as UTF-8 without BOM. 