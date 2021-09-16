---
title: Fibonacci
author: Liam McLennan
date: 2012-02-15 20:32
template: article.jade
---

Here is the fibonacci sequence calculated in haskell (1 to 100):

```haskell
fibo :: Int -> Int
fibo 0 = 0
fibo 1 = 1
fibo n = (fibo (n-1)) + (fibo (n-2))

prn = map fibo [1..100]

prn
```

and here it is in CoffeeScript

```coffeescript
fibo = (n) ->
  if n is 0 or n is 1
    return n

  fibo(n-1) + fibo(n-2)

for i in [1..100]
    console.log fibo(i)
```

Both of these take forever to execute. Curiously the CoffeeScript version is an order of magnitude faster. 