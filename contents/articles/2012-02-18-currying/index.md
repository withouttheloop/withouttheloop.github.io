---
title: Currying
author: Liam McLennan
date: 2012-02-18 15:00
template: article.jade
---

This is a function that does some currying:

```coffeescript
add = (a,b) ->
  if not b?
    return (c) ->
      c + a
  a + b
```

JavaScript provides the capability to reflect on the number of arguments:

```coffeescript
add.length
```

and to determine how many arguments were provided:

```coffeescript
add = (a,b) ->
  if arguments.length < add.length
    return (c) ->
      c + a
  a + b
```

so it seems like it should be possible to write a function that magically returns a function that requires the right number of arguments. So I could have a function:

```coffeescript
f = (a,b,c,d,e,f) ->
  ..
```

if invoked with:

```coffeescript
f(1,2)
```

it should return:

```coffeescript
(c,d,e,f) ->
```

anyone know how to do that?

> Update

> Lots of good comments on [the original gist](https://gist.github.com/liammclennan/3654718)