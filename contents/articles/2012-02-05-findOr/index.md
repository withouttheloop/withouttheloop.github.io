---
title: Extending Underscore.js - findOr
author: Liam McLennan
date: 2012-02-05 20:32
template: article.jade
---

As good as [underscore.js](http://underscorejs.org) is, sometimes you want to add functions of your own, and you can, using the [mixin() function](http://underscorejs.org/#mixin). 

    _.mixin({
      capitalize : function(string) {
        return string.charAt(0).toUpperCase() + string.substring(1).toLowerCase();
      }
    });
    _("fabio").capitalize();
    => "Fabio"

findOr
------

Underscore has a find method that returns the first element of a collection that matches a supplied predicate. For example, it can be used to find the first even number in a set:

    var isEven = function (num) {
      return num % 2 === 0;
    }

    _([1,2,3,4,5]).find(isEven);
    => 2

findOr is a function I mixed-in to underscore so that I could have find return a default value if no element matches the predicate.

    _([1,2,3]).findOr(isEven, 'no even numbers');
    => 2

    _([1,3]).findOr(isEven, 'no even numbers');
    => 'no even numbers'

The implementation, using underscore's mixin function, is:

    _.mixin({
      'findOr': function(list, iterator, defaultValue, context) {
        return this.find(list, iterator, context) || defaultValue;
      }
    });

Try the executable demo on [jsfiddle](http://jsfiddle.net/ynkJE/22/).