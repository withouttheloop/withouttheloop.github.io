---
title: Haskell influences in CoffeeScript
author: Liam McLennan
date: 2013-02-27 20:32
template: article.jade
---

While learning a bit of Haskell I was struck by the syntactical similarites to CoffeeScript. Since Haskell predates CoffeeScript by twenty years (1990/2010) it seems that it is Haskell that has had an influence on CoffeeScript. What follows is a list of the similarities that I have observed.

Binding Variables
---------------

In Haskell we can bind a local variable to a scope using the `let... in` syntax:

```
circumference r = let pi = 3.14159
                  in  pi * 2 * r
```

CoffeeScript supports the same thing via the `do` keyword. One interesting thing about `do` is that it allows a way to define a variable that is (sort of) not scoped to a function.

```
circumference = (r) ->
  do (pi = 3.14159) ->
    pi * 2 * r
```

You can read about some interesting uses of `do` in Reginald Braithwaite's [CoffeeScript Ristretto](https://leanpub.com/coffeescript-ristretto).

Significant Whitespace
----------------------

As you can see in the examples above Haskell and CoffeeScript both use significant whitespace in a similar way. 

Expressions > Statements
------------------------

Both Haskell and CoffeeScript encourage us to favour expressions over statements. CoffeeScript supports statements that aren't expressions but does [everything possible to let you avoid them](http://coffeescript.org/#expressions).

```
evenness = (i) ->
  if i % 2 is 0
    'even'
  else
    'odd'
```

Combining this with the `do` notation we can do

```
evenness = (i) ->
  do (is_even = (n) -> n % 2 is 0) ->
    if is_even i
      'even'
    else
      'odd'
```

Comprehensions
--------------

Haskell and CoffeeScript have similar syntax for list comprehension. The following Haskell function selects the even elements of a list (`x mod 2 == 0`) and maps them through a function that multiplies them by two (`x*2`).

```
double_odds xs = [x*2 | x <- xs, x `mod` 2 == 0]
```

The equivalent CoffeeScript is:

```
double_odds = (xs) ->
  x*2 for x in xs when x % 2 is 0
```

Function Call Syntax
------------------

Both languages use a parenthesis free syntax for applying a function. 

Haskell:

```
add 2 3
```

CoffeeScript:

```
add 2, 3
```

Ranges
------

For basic incrementing or decrementing integers Haskell and CoffeeScript have the same syntax.

```
[1..10]

[10..1]
```

Literate Mode
-------------

Haskell and CoffeeScript both support a 'literate' mode that emphasizes comments over code. 

In [CoffeeScript's literate mode](http://coffeescript.org/#literate) markdown text is interpreted as a comment. Indented text is executed as CoffeeScript code. 

```
Heading
=======

What I need is a function that doubles the even numbers in a list. Here's one!

  double_odds = (xs) ->
    x*2 for x in xs when x % 2 is 0
```

Haskell's literate mode uses a `>` to indicate lines of code.

```
What I need is a function that doubles the even numbers in a list. Here's one!

> double_odds xs = [x*2 | x <- xs, x `mod` 2 == 0]
```

That's all that I can think of, but I'm sure there is more. CoffeeScript is often described as a blend of Ruby and Python that compiles to JavaScript. I think this underplays the influence of Haskell in CoffeeScript's design.