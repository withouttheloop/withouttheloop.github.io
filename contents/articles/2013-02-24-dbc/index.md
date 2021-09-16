---
title: dbc (node.js npm module)
author: Liam McLennan
date: 2013-02-24 15:00
template: article.jade
---

dbc
===

[dbc](https://npmjs.org/package/dbc) is my [third npm module](https://npmjs.org/~liammclennan). I am experimenting with the node.js (unix?) philosophy of small, composable modules. dbc provides some basic tools for [design-by-contract](http://en.wikipedia.org/wiki/Design_by_contract), something that I believe is essential when working in a loosely typed language like JavaScript. 

To install:

```
    npm install dbc
```

to run the tests:

```
    npm tests/ -R spec
```

which will produce this delightful output:

<img src="/articles/2013-02-24-dbc/images/dbc.png" alt="dbc test output" />