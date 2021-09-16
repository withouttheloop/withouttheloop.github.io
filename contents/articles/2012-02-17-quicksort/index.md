---
title: Quicksort in Haskell and CoffeeScript
author: Liam McLennan
date: 2012-02-17 15:00
template: article.jade
---

The definition of Quicksort, from Wikipedia is:

> The steps are:

> 1. Pick an element, called a pivot, from the list.
> 2. Reorder the list so that all elements with values less than the pivot come before the pivot, while all elements with values greater than the pivot come after it (equal values can go either way). After this partitioning, the pivot is in its final position. This is called the partition operation.
> 3. Recursively sort the sub-list of lesser elements and the sub-list of greater elements.

> The base case of the recursion are lists of size zero or one, which never need to be sorted.

Quicksort in Haskell, from learn you a haskell, is:

```haskell
    quicksort :: (Ord a) => [a] -> [a]  
    quicksort [] = []  
    quicksort (x:xs) =   
        let smallerSorted = quicksort [a | a <- xs, a <= x]  
            biggerSorted = quicksort [a | a <- xs, a > x]  
        in  smallerSorted ++ [x] ++ biggerSorted 
```

A basic quicksort in CoffeeScript is:

```coffeescript
quicksort = ([head,tail...]) ->
  return [] if typeof head is 'undefined'
  smaller_sorted = quicksort (e for e in tail when e <= head)
  larger_sorted = quicksort (e for e in tail when e > head)
  smaller_sorted.concat([head]).concat(larger_sorted)  
```

The CoffeeScript version is reasonable, and shares some similarity with the haskell implementation. The part I don't like is having to check `typeof head is 'undefined'` for the base case. This is because the destructuring assignment of the function argument leaves me with no reference to the full array so the only way to tell if it is empty is to check if the head is undefined. There is probably a better way to do this. Please comment if you know what that is.