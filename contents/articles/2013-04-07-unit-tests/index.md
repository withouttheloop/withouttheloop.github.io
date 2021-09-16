---
title: Can we stop saying 'unit tests'?
author: Liam McLennan
date: 2013-04-07 20:32
template: article.jade
---

What is a unit test?
--------------------

Here is wikipedia's definition of unit testing:

> In computer programming, unit testing is a method by which individual units of source code, sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures, are tested to determine if they are fit for use.

Looking carefully we notice that, "units of source code, sets of one or more computer program modules" is actually everything. Therefore, the above definition can be condensed, without changing its meaning, to:

> In computer programming, unit testing is a method by which any source code is tested to determine if it is fit for use.

That is, unit testing is testing. Great.

Roy Osherove's definition is equally vague and pointless:

> A unit test is a piece of a code (usually a method) that invokes another
piece of code and checks the correctness of some assumptions after-
ward. If the assumptions turn out to be wrong, the unit test has failed.
A “unit” is a method or function.

Prior to the last sentence this was a useful definition of an automated test, but the last sentence, "a unit is a method or function" is seemingly unrelated to the rest of the definition. Defining a "unit" does not help unless we choose to infer that a "unit test" is a test that tests a "unit". If a "unit test" is a test that tests a "unit", and a "unit is a method or function" then if we factor our programs into cooperating functions, as suggested by [functional decomposition](http://en.wikipedia.org/wiki/Decomposition_%28computer_science%29), the [composed method pattern](http://c2.com/ppr/wiki/WikiPagesAboutRefactoring/ComposedMethod.html) and the [SOLID principles](http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29) then it is impossible to write a "unit test" according to Osherove's definition, since any function that calls another function is by definition not a "unit". Testing a function that depends upon other functions is obviously testing all of the functions involved. 

Despite a lot of pretense the fact is that we do not have an accepted definition of a unit test. 

What is the harm?
-----------------

I have always treated "unit test" as the antonyn of "integration test". This has been a useful dichotomy. Unit tests are fast, integration tests are slow therefore it is useful to separate them. 

The ambiguity of the term only becomes a problem when people malign tests with statements such as, "that's not a unit test". That may well be, or it may not be, it's hard to say since there is no consistent, useful definition of a unit test.

"Unit tests are good" is an empty bit of dogma that has outlived is usefullnes, finally becoming not just useless but harmful. 