---
title: Maybe Haskell?
author: Liam McLennan
date: 2013-05-19 20:32
template: article.jade
---


Usual disclaimer - I don't know Haskell at all. What follows is my rambling experimentation with the `Maybe` type.

Haskell has [a useful type called `Maybe`](http://www.haskell.org/ghc/docs/6.12.2/html/libraries/base-4.2.0.1/Data-Maybe.html). F# and probably most other functional languages have something similar. Even C# has Nullable<T> which is somewhat in similar in that its purpose is ["to encapsulate an optional value"](http://www.haskell.org/ghc/docs/6.12.2/html/libraries/base-4.2.0.1/Data-Maybe.html). To put it another way, `Maybe` is a parameterised or polymorphic type that represents that we possibly have a value of the type parameter. So `Maybe Int` means maybe we have an `Int`, maybe we don't (ie we have Nothing).

Because Haskell doesn't allow `null` values we might use `Maybe` for a function that returns its input if the input is a positive number.
    
    giveIfEvan :: Int -> Maybe Int
    giveIfEvan n = if n `mod` 2 == 0
                    then Just n 
                    else Nothing

`Maybe` is a monad, which, to me, means primarily that it provides a function to convert from a `Maybe` of some type to a `Maybe` of some other type (`>>= :: Maybe a -> Maybe b`).

Imagine I want a function that given a `Maybe Int` adds 1 to the value (if there is a value). One terrible way to write this is:

    addOne :: Maybe Int -> Maybe Int
    addOne n = if isJust n
                then Just (fromJust n + 1)
                else Nothing

or with pattern matching:

    addOne :: Maybe Int -> Maybe Int
    addOne (Just a) = Just (a + 1)
    addOne Nothing = Nothing

since `Maybe` is a monad we can use do notation:

    addOne n = do
            v <- n
            return (v + 1)

The benefit here is that `addOne` handles the nothing case without an explicit conditional. 

If you don't mind using a lambda you can use the de-sugared version of do `>>=` (bind):

    addOne n = n >>= \v -> return (v + 1)

Or instead of a lambda we could define an extra function:

    increment :: Int -> Maybe Int
    increment v = return (v + 1)

    addOne :: Maybe Int -> Maybe Int
    addOne n = n >>= increment

Here is a complete program. Try changin the 8 to an odd number to see the nothing case.

    import Data.Maybe

    giveIfEvan :: Int -> Maybe Int
    giveIfEvan n = if n `mod` 2 == 0
                    then Just n 
                    else Nothing

    addOne :: Maybe Int -> Maybe Int
    addOne n = do
                v <- n
                return (v + 1)

    main = do
      return (addOne $ giveIfEvan 8)
