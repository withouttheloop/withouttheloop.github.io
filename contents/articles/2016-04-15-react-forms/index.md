---
title: React Forms
author: Liam McLennan
date: 2016-04-15 20:32
template: article.jade
---

<img src="spider.jpg" alt="spider (Argiope bullocki)" style="width:300px" align="right"/>

As I have [said before](http://withouttheloop.com/articles/2014-05-29-react-flux-etc/), React focuses on mapping user interface state to the DOM and publishing events. It doesn't try to help with common application tasks like routing, http requests, or managing forms. So how do we work with forms in React?

The Basic React Form Pattern
============================

The design of React leads to a curious contradiction for forms. 

> If the user interface is a transformation of the user interface state then it should not be possible to modify the content of a form without updating the application state

And this is exactly where we start with React forms. Here is the most basic form implementation possible. The application model is bound to the form fields. Note that it is not possible for the user to modify the content of the form (see the 'Result' tab). It would not make sense because the form is supposed to be a reflection of the application state. 

<iframe width="100%" height="300" src="//jsfiddle.net/69z2wepo/38770/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

The way to solve this problem is to break one of my own rules of working with React - we let the component maintain local state. The risk here is that the content of the form could become inconsistent with the application state. For the content of a form, this is something that I can live with. The above example, modified to use local component state for the content of the form is:

<iframe width="100%" height="300" src="//jsfiddle.net/69z2wepo/38772/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

The form fields are no longer bound to the components properties. Instead they are bound to local state, that is copied from the components properties (in the `getInitialState()` lifecycle method). That provides one-way binding from the state to the form. To bind changes to the form back to the component state we need the `bindState` method. `bindState` creates event handlers that push changes to the form back to the component state. Everytime the user modifies a character in a form field it is pushed back to the component state, which causes the component to re-render with the new form content. We now have a form that allows the user to change form field values. 

We can then add a form submission handler to deal with changes to the form when it is submitted. 

<iframe width="100%" height="300" src="//jsfiddle.net/69z2wepo/38773/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Because we have been synchronizing the form to the component state on every character change we already have the latest content of the form in the state. So what do we do with it?

What To Do With Form Submissions
================================

There are two ways to handle form submission. Which you chose depends on the type of component. If I was building a reusable widget (like a calendar or something) then I would express all results as calls to a property of the component.

<iframe width="100%" height="300" src="//jsfiddle.net/69z2wepo/38774/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Note how `onSubmit` is a callback function passed to the `PersonForm` component as a prop. When the form is ready to publish its result it simply calls the supplied `onSubmit` function and passes its result. 

This approach has elegance. Components designed in this way are perfectly composable. The flipside is that components exist in a heirarchy, and results communicated via callbacks must be bounced all the way up the heirarchy to the top. Applying this strategy everywhere is a productivity and maintenance nightmare, leading to the rule:

> use callbacks to communicate results only for re-usable, widget-style components

For other components, publish results out of your React component heirarchy to a state management solution like [redux](http://withouttheloop.com/articles/2015-12-28-tetris3/) to merge the changes into your application state.

<iframe width="100%" height="300" src="//jsfiddle.net/69z2wepo/38776/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Note that the value passed to `dispatch` is an object of the form `{ type: '', ... }`. This is a redux convention. The general idea of libraries like redux/flux etc is that they take these actions, update the application state and re-render the entire application component graph, preserving the symmetry between the application state and the DOM. 

Higher-level Form Abstractions
==============================

A great way to avoid this issue all together is to use one of many libraries providing higher-level form abstractions. Typically, the library generates an appropriate form based on a supplied piece of state. Such libraries are terrific for productivity but are at their best when you do not require precise control over the form. Once you start having to invest effort in customizing how a tool generates forms for you I think you are better off reverting to building the form yourself.  

Validation
==========

I wrote recently about [JavaScript object validation with JSON Schema](http://withouttheloop.com/articles/2016-04-14-javascript-object-validation/) so I have covered how to validate objects. The form approach described above covers how to bind objects to and from forms with React. The missing piece then is just how to trigger validation and handle displaying any validation errors to the user. I will investigate this in a future post. 

