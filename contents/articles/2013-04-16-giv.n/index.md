---
title: giv.n
author: Liam McLennan
date: 2013-04-16 20:32
template: article.jade
---

Today I published my [first nuget package](https://github.com/liammclennan/giv.n) - a ridiculously simple BDD library written by [Nicholas Blumhardt](http://nblumhardt.com/) and I. 

giv.n is a tiny DSL for grouping the steps of an executable specifications into the familiar given, when and then (or `giv.n`, `wh.n` and `th.n` in giv.n language). It does not have a test runner or an assertion library so you still need NUnit or similar. Here is an example specification written with giv.n and NUnit.

```
using Givn;

[Test]
public WhenIBakeACake() {
    Giv.n(IHaveFlour)
        .And(IHaveEggs);
    Wh.n(IBakeCake);
    Th.n(IHaveDeliciousSnack);
}

private void IHaveFlour() {
    _ingredients.Add(new Flour());
}

private void IHaveEggs() {
    _ingredients.Add(new Eggs());
}

private void IBakeCake() {
    _cake = _oven.Bake(_ingredients);
}

private void IHaveDeliciousSnack() {
    Assert.IsTrue(_cake.IsDelicious());
}
```
