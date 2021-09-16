---
title: Testing backbone models
author: Liam McLennan
date: 2012-02-13 20:32
template: article.jade
---

This is this first in a planned series discussing the specifics of testing applications built with Backbone.js. 

Of all the components of a Backbone.js application Models are the easiest to test because they don't have a lot of dependencies. The high level testing approach I use is:

1. Initialize a model with a specific state

2. Test that the model’s behavior matches expectations

3. Goto 1.

Testing a Rectangle Model
-------------------------

Imagine a Backbone model that represents a rectangle, with length and width properties. It has the following specification:

### Rectangle Specification

> Rectangle  
>>    with length 7 and width 4  
>>>        should have an area of 28  
>        should have a perimeter of 22  

### Jasmine Specification

We can express the rectangle specification using the syntax of the [Jasmine testing tool](http://pivotal.github.com/jasmine/):

    describe('Rectangle', function () {
      describe('with length 7 and width 4', function () {
        it('should have an area of 28', function () { 
          // assert expectations here
        });
        it('should have a perimeter of 22', function () { 
          // assert expectations here
        });
      });
    });

and add the expectations to get a failing test:

    describe('Rectangle', function() {
        var rectangle;

        beforeEach(function() {
            rectangle = new app.Rectangle();
        });

        describe('with length 7 and width 4', function() {
            beforeEach(function() {
                rectangle.set({
                    length: 7,
                    width: 4
                });
            });

            it('should have an area of 28', function() {
                expect(rectangle.area()).toBe(28);
            });
            it('should have a perimeter of 22', function() {
                expect(rectangle.perimeter()).toBe(22);
            });
        });
    });

The Rectangle Model
-------------------

The final step is to implement the Rectangle model to make the tests pass:

    var app = {};

    (function (shapes) {
        shapes.Rectangle = Backbone.Model.extend({
            area: function () {
                return this.get('length') * this.get('width');
            },
            perimeter: function () {
                return 2*this.get('length') + 2*this.get('width');
            }
        });
    })(app);

Et Voilà!
--------

Testing Backbone models is trivial. Next in this series I will describe how I test Backbone views - a much more challenging proposition.