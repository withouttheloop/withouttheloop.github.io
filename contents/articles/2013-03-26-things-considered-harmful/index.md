---
title: Things considered harmful (in C#)
author: Liam McLennan
date: 2013-03-26 20:32
template: article.jade
---

Spelunking through a program recently I started to think about my coding style, particularly in C#. It occurred to me that my style is most easily defined in terms of the things that I try to avoid. Here is a quick list of things to avoid, in C#, where practical.

Most of the justification for these exclusions is in my presentation [Why Functional Programming Matters](https://dl.dropbox.com/u/8948049/fp/dddbrisbane2012-master/reveal.js/index.html)

Loops
----

I don't like explicit loops, for and foreach. Use linq when querying. Explicit loops are better when mutating state but let's avoid that too. Recursion is also an option. 

Conditional (if)
------------

Null checking type conditionals can be replaced with null coallescion in a variant of the null object pattern:

    (callback ?? ()=> {})();
    
Polymorphism allows types to replace explicit conditionals. Tell don't ask can also eliminate conditionals. Finally, the conditional operator (?:) is preferable to the if statement because it is an expression.

Switch statement 
---------------

Switch is a particularly nasty special case of conditionals.

Avoid by using polymorphism or convert to a dictionary (declarative instead of imperative). 

Statements 
------------------

C# make these hard to avoid because many C# language feature are not expressions. The solution is to use language features that are expression oriented, like linq, factor functions properly and write pure functions. Variable declarations are ok if they make code more readable.

Mutable state
------

Don't have it if you don't need it. Return copies of things.

State mutations
----------

As above.

async
----

The TPL is great when you need to do something asynchronously, but it is viral in the sense that all callers become async too. Don't use async/TPL without a good reason and do it as shallow in the callstack as possible. Separate async / IO code from pure code 

null
----

Null is [Tony Hoare](http://en.wikipedia.org/wiki/Tony_Hoare)'s fault. 

> I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

Eric Lippert seems to agree:

> even this is essentially an accident of history; it just so happened that when C# was first implemented it had always-nullable reference types, non-nullable value types, and no generic types at all. In a counterfactual world where the CLR had generic types from the get-go, it seems plausible that Nullable<T> could have been implemented to work on any type, and reference types would then be non-nullable by default. We could have a type system where Nullable<string> was the only legal way to represent "a string that can be null". Keep this in mind the next time you design a new type system!

Give references default values and design types that don't contain null members. No more null reference exceptions.

ref and out params
------------------

It is often (always?) possible to replace ref and out params with other options, such as pre-testing values. 
