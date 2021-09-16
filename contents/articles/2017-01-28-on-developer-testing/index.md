---
title: On Developer Testing
author: Liam McLennan
date: 2017-01-28 20:32
template: article.jade
---

On Developer Testing
===================

Why
---

A developer's responsibility is to create software that works. 

> If it has not been tested it has not been shown to work. 

As part of the planning for implementing a new feature the developer should think about and plan how the feature will be tested, and have agreement from the product owner (or appropriate stakeholder). This is sometimes called documenting acceptance criteria.

[More information on developer testing](http://web.archive.org/web/20130729213428id_/http://itc.conversationsnetwork.org/shows/detail301.html).

Test Design Prior to Implementation
----------------------

Acceptance criteria may be recorded in the classic BDD syntax:

```gherkin
GIVEN some preconditions
WHEN an action happens
THEN a result occurs
```

For example,

```gherkin
GIVEN an authenticated user
WHEN they submit the enquiry form
THEN the contents of their enquiry is sent to info@acme.com
AND a copy is stored in the database
```

Repeat until all scenarios have been described. This process ensures: that developers and their stakeholders share an understanding of what is to be built, that the developer has thought through the alternatives that must be accounted for in the implementation. 

Test Implementation
------------------

Theoretically, tests should exist for all enumerated scenarios. In practice some may be difficult to implement or may have insufficient value to justify their existence. Having tests for all scenarios provides a convenient scaffold for the developer. They begin with a complete set of unimplemented tests, make progress by implementing the scenarios, and have confidence that the implementation task is complete when all scenarios have passing tests. 

Good tests should only fail because the code they test is wrong. Therefore, most tests should be ['unit' tests](http://withouttheloop.com/articles/2013-04-07-unit-tests/). Integration tests can fail for many reasons so they are undesirable. However, the value of a small number of integration tests is extremely high, as they show that the system functions together as a unit. Start with a small number of end-to-end integration tests, then favour small, focused unit tests. 

Code for Testing
---------------

Write code that is easy to test. The easiest code to test is code that transforms an input to an output, e.g. a function. Separate computation from side effects, and test the computation (testing side effects pushes you into an integration test). Although this is counterintuitive it leaves more testable and maintainable code that is also more reusable (via composition). It is a happy coincidence that good code and testabe code happen to be the same thing.

Here is an example of how computation and side effects are commonly mixed:

```javascript
if (new Date().getHours() > 11) {
    console.log("Good afternoon");
} else {
    console.log("Good morning");
}
```

Accessing the current date via `new Date()` is a side effect that introduces a hidden variable that a test cannot control. Producing a result via `console.log` similarly prevents a test from evaluating behaviour. The computation here is choosing a greeting based on the time of day, so extract that and test it.

A better version is:

```javascript
// this is a testable function
function chooseGreeting(hourOfDay) {
  let isAfternoon = hourOfDay > 11;
  return isAfternoon 
    ? "Good afternoon"
    : "Good morning";    
}

// group the side effects together
console.log(chooseGreeting(new Date().getHours()));
``` 

Test Execution
-------------

The goal is to know as early as possible when a mistake has been made. This is one reason why a good type system is surperior to any testing. I like to say,

> every test is an admission of failure.

What I mean is that any other technique for preventing errors is surperior to testing. This could be a static type checker, design-by-contract, transactions or some particular design. When those techniques have been exhausted then we must fallback to testing. 

The strength of testing is its flexibility to detect any type of errors (not just type errors). The weakness of testing is its inability to give any information about anything other than the exact scenario tested (hence the power of [property testing](https://en.wikipedia.org/wiki/QuickCheck)). 

To receive the test results as early as possible it is helpful to have a system that runs the tests when changes are detected that may change the test results. Since tests are primarily a function of source code this is usually implemented via a file system watcher that detects changes to the tests or the code that they test, and reruns the tests. 

Each developer should check the tests, and not commit code that causes a test to fail, however, human discipline is flawed and we should only rely upon it when there is no alternative. In the case of testing the usual solution is to make test execution part of the continuous integration build process triggered by every code commit to the shared main branch, which should happen at least once every day. This guarantees that all deployment candidates have passed all tests. 



