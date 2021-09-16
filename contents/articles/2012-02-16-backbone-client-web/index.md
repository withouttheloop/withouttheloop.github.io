---
title: Backbone.js and Client-side Web Applications
author: Liam McLennan
date: 2012-02-16 15:00
template: article.jade
---

Backbone.js and Client-side Web Applications
============

In the beginning the internet was a set of documents. Then someone decided to add hyperlinks to create a web of connections between documents. Soon static documents were not enough so we began to dynamically create documents on the server. The client-side user experience was still flat, so scripting was introduced to add dynamic behavior to pages. Finally, the most demanding and advanced web applications moved their user interface entirely to the client-side to provide the ultimate user experience. This apocryphal tale demonstrates the long, unbroken trend towards richer user interfaces on the web. 

The new generation of frameworks, libraries and training material make developing client-side web applications fun and approachable. 

> Warning: the rest of this article assumes some knowledge of JavaScript. If you would like to learn more about JavaScript I have recorded a [comprehensive JavaScript introduction screencast](http://pluralsight.com/training/Courses/TableOfContents/jscript-fundamentals) for pluralsight.

Backbone.js
-----------

Backbone.js is a [delightfully documented](http://backbonejs.org/) set of tools that help us to simplify and structure client-side web applications. 

Backbone provides the building blocks for assembling client-side applications. Models are used to store an application's data and to handle sending data to and from the server. Views map to user interface elements. They render markup, usually based on models, and connect event handlers to document events. Finally, backbone's router is used to trigger and handle client-side routing. 

Putting these basic elements together we can assemble complex applications in an enjoyable and productive way. The rest of this article will investigate a simple application that makes use of the model and view components of Backbone.js. 

For a complete introduction to Backbone.js, including guidance on how to structure and test backbone.js applications, try my [Backbone Fundamentals screencast](http://pluralsight.com/training/Courses/TableOfContents/backbone-fundamentals) course.

The Basics
----

Backbone.js does many things - one of the most immediately useful is the ability to bind one data object to multiple user interface components. When the underlying data model is changed each of the views is automatically updated to reflect those changes. 

Imagine a user interface that displays a color as its hex value, and as a swatch. The color can be represented as a backbone model. The hex display and the swatch are each a backbone view. 

<img alt="mockup" src="https://dl.dropbox.com/u/8948049/backbone/mockup.png" />

[Try the completed application](http://jsfiddle.net/ynkJE/171/) but ignore the code for now. 

The single color model can be shared between the two views. That way there is a single authoritative data source, and no possibility of synchronization problems. 

<img alt="class diagram" src="https://dl.dropbox.com/u/8948049/backbone/static.png" />

A first Backbone.js Model
------

The data object (backbone model) can be created like this:

```js
var color = new Backbone.Model({
  hex: '#ff0000'
});
```

For a starting point I have set the model's hex value to `#ff0000` which is the hex code for red.

Backbone.js Views
------

The hex view, which displays the hex value of the color in a text box is: 

```js
var HexView = Backbone.View.extend({
    render: function () {
        // create a textbox containing the model's hex attribute
        var input = this.make('input', { type: 'text', value: this.model.get('hex') });
        this.$el.append(input);
    }    
});
```

`render()` is the function that backbone uses to update a view's markup. `this.model` is the view's reference to the shared color model.

The swatch view, which displays a color swatch representing the color model, is implemented as:

```js
var SwatchView = Backbone.View.extend({
    render: function () {
        this.$el.html(this.make('div', { class: "swatch", style: "background-color: " + this.model.get('hex') }));
    }
});
```
see the [full source](http://jsfiddle.net/ynkJE/170/)

To illustrate the real power of Backbone.js we need to change the model data, and trigger an update of the views. To do this, we add a button to the hex view and handle its click event.

```js
var HexView = Backbone.View.extend({
    // Every view type can have an events property that declares event bindings.
    events: {
        'click input': function () {
            this.model.set({hex: this.$('input').val()});
        }
    },
    
    render: function () {
        var input = this.make('input', {type: 'text', value: this.model.get('hex')});
        this.$el.html(input);
        this.$el.append(this.make('input', {type: 'button', value: 'update'}));
    }    
});
```

When the button is clicked the view updates the model with the color currently in the text box. To get this change to show up in the swatch view we need to bind the model's `change` event to the swatch view's `render()` function. 

```js
var SwatchView = Backbone.View.extend({
    initialize: function () {
        // handle the model's `change` event
        this.model.on('change', function () {
            this.render();
        }, this);
    },
    
    render: function () {
        this.$el.html(this.make('div', { class:"swatch", style:"background-color: " + this.model.get('hex') }));        
    }
});
```

`initialize()` is a special view function that is run when an instance of the view is created (like a constructor).

Connecting the dots
-------------

We have a model object (`color`) and two view types (`HexView` and `SwatchView`). To initialize our application we just need to create instances of the views and call the `render()` methods. 

```js
var hex = new HexView({model: color, el: '#hex'});
hex.render();

var swatch = new SwatchView({model: color, el: '#swatch'});
swatch.render();​
```

[As you can see](http://jsfiddle.net/ynkJE/171/) I can now enter a different hex code (say `#0000ff`), click the update button and see the swatch change to blue.  

<img src="https://dl.dropbox.com/u/8948049/backbone/blue.gif" alt="complete backbone.js interface" />
<br/>The completed user interface

That concludes our exploration of the basic features of Backbone.js. If you are interesting in learning more about Backbone.js and building client-side web applications try For a complete introduction to Backbone.js, including guidance on how to structure and test backbone.js applications, try my [Backbone Fundamentals screencast](http://pluralsight.com/training/Courses/TableOfContents/backbone-fundamentals) course.