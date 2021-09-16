---
title: Reversing a string
author: Liam McLennan
date: 2013-02-20 20:32
template: article.jade
---

While there are many ways to skin a cat I can only think of the one involving a knife. However, I can think of many ways to reverse a string. 

Recursive function
------------------

My preferred solution is a simple recursive function (gistfile1.cs). The first character of the reversed string is the last character of the input. The rest of the string is the rest of the input reversed. The only other thing required is to detect when the entire string has been processed and terminate the recursion.

Map from a collection of indexes
--------------------------------

Create a collection of indexes and map each one to its inverse location in the string (gistfile2.cs).

The CoffeeScript version (gistfile3.coffee) has nicer syntax. It also demonstrates the affect of reversing the sequence of indexes.

<script src="https://gist.github.com/liammclennan/4994673.js"></script>