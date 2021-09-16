---
title: Reading Code - robots-txt 
author: Liam McLennan
date: 2013-05-30 20:32
template: article.jade
---

<img src="/articles/2013-05-30-reading-code/cave.jpg" align="left" style="margin: 0px 20px 20px 0px;" alt="spelunking"/> Scott Hanselman has or had a series called [the weekly source code](http://www.hanselman.com/blog/TheWeeklySourceCode59AnOpenSourceTreasureIronyNETLanguageImplementationKit.aspx), in which he reviewed open source code as a learning exercise. I have enjoyed looking through the source of [robots-txt](https://github.com/meanpath/robots) by [Mark Wotton](https://github.com/mwotton) so I will follow Mr Hanselman's lead and document my experience spelunking this source.

robots-txt.cabal
----------

The first thing I noticed, in the root of the repository, is the robots-txt.cabal file. This file defines how the project will be packaged for use as a dependency of other projects and also the dependencies of the current project for installation. robots-txt is defined as a library, meaning that it exports things to be used in other applications and does not have its own executable. 

Below the standard package definition robots-txt.cabal includes a setting I have not seen before, [test-suite](http://www.haskell.org/cabal/users-guide/developing-packages.html). robots-txt uses the older exitcode-stdio-1.0 + main-is style of test suite 'that indicate test failure with a non-zero exit code'. Adding the test-suite configuration means that the project's tests can be run with `cabal test`. Yet another reason to add cabal configuration to all projects, not just the ones that are intended to be distributed as cabal packages.

The usage of `hs-source-dirs` for both the library and test-suite configuration is also interesting. 

Tests
-----

The robots-txt.cabal file defines the entry point for testing as test/Spec.hs. Spec.hs references [hspec-discover](https://github.com/sol/hspec-discover) in a GHC_OPTIONS pragma. hspec-discover looks for and executes all tests in the directory and any subdirectories. This is a convenient way to avoid having to maintain a list of tests to execute. In the case of robots-txt there is only one test file to discover test/RobotSpec.hs.

### RobotSpec.hs

[RobotSpec.hs](https://github.com/meanpath/robots/blob/master/test/RobotSpec.hs) uses [hspec](http://hspec.github.io/) and the specification style of tests to define the behaviour of the project. It is curious that this style (describe... it... should) is popular on so many platforms and yet I have not seen a .NET implementation. 

RobotSpec.hs uses the `{-# LANGUAGE CPP #-}` pragma, which allows it to use the special symbol __FILE__ which is presumably replaced by the current filename pre compilation. 

I don't understand how RobotSpec is executed, since it doesn't declare a main function like the hspec examples and it doesn't export anything. 

At first I didn't understand the use of forM_, but a bit of googling, and particularly [this stackoverflow question](http://stackoverflow.com/questions/5856709/what-is-the-difference-between-liftm-and-mapm-in-haskell) provided some insight.

### Network.HTTP.Robots

The actual program is contained within the [Network.HTTP.Robots module](https://github.com/meanpath/robots/blob/master/src/Network/HTTP/Robots.hs). It defines a number of data structures and some functions for operating on them. Some of it is confusing to me, such as the directiveP function. I suspect I need to spend some more time with Control.Applicative

Conclusion
----------

Looking through Robots has been both fun and enlightening. The code is simple enough for me to understand (mostly) and it is small enough to be easily digestable. I've learnt a few things, particularly about how to setup testing which is an area that I have been planning to improve. 