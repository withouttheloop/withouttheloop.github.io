---
title: Backbone app framework
author: Liam McLennan
date: 2012-02-12 20:32
template: article.jade
---

Backbone is a library of tools that simplify the design and implementation of client-side web applications. It is explicitly not a framwework. Backbone does not provide guidance about how to assemble an application. This post will be an initial attempt at filling that gap. It assumes an intermediate level of Backbone.js knowledge. If you have never used Backbone.js try [Backbone.js Basics](http://hackingon.net/post/Backbonejs-Basics.aspx) or the [Backbone.js homepage](http://backbonejs.org/).

The Application Root
--------------------

I like to provide a root object/namespace for my application. Let's call it `app`.

    var app = {};

Register Components on the Application Object
--------------------------------------------

    app.models =  {
        Post: Backbone.Model.extend({}),
        Comment: Backbone.Model.extend({})
    };

    app.views = {
        PostView: Backbone.View.extend({}),
        ComentView: Backbone.View.extend({})
    };

De-couple Components by using an Event Aggregator
-----------------------------------------------

An application event aggregator allows components to communicate indirectly. A simple event aggregator can be created by mixing the Backbone.Events modules into an application level object:

    app.eventAggregator = _.extend({}, Backbone.Events);

Actions
-------

When a significant event occurs within your application raise an event. For example, the handler for the 'add post' button:

    _onAddPost: functin () {
        app.eventAggregator.trigger('post:create', this.model);
    }

Register an 'action' to handle the application event:

    app.actions = {
        createPost: Action.extend({
            event: 'post:create',
            handler: function (model) {
                var newPost = new app.Views.CreatePost({model: model});
                $('#content').html(newPost.render().el);
            }
        })
    };

Routing
-------

Use a route only when you want to change the url and handle the event. Otherwise just raise an event and register an action.

When you do use a route, have it trigger an action. 

    app.routers = {
        postRouter: Backbone.Router.extend({
            'view/:postname': function (postname) {
                app.eventAggregator.trigger('post:view', postname);
            }
        })
    };

One Line Bootstrapping
----------------------

Reduce your application bootstrapping to a single line.

    app.start();

This makes it practical to test contexts that require the application to be in a usable state.

    describe('adding a new post', function () {
        describe('initiate new post process', function () {
            before(function() {
                app.start();
                var testModel = new app.models.post();
                app.eventAggregator.trigger('post:create', testModel);
            });
        });
    });

The `start()` function is the framework part. It registers the routers and actions and does any other required bootstrapping. 