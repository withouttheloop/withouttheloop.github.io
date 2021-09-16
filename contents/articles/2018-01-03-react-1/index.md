---
title: Learning React 1 - A first component
author: Liam McLennan
date: 2018-01-01 20:32
template: article.jade
---

While doing some maintenance on my Pluralsight course, [React Fundamentals](https://www.pluralsight.com/courses/react-fundamentals) I have been thinking about how to teach good front-end development skills. At first I was puzzled as to why people struggled, when React is fundamentally very simple. 

To build a complete application, React is not enough. You have to add supporting tools to help with routing, bundling, state management and other services not provided by React itself. These additional tools add a lot of complexity, and they are presented to learners in their final, complete and extremely confusing way. Between the React 'Hello World' demo and a completed, ready-for-production application there is a chasm that seems to be ignored by the standard documentation. 

In this post, and more to follow I intend to show how a React application is built up from first principles. I will try to add complexity gradually and explain why the different pieces are necessary. Let's begin with a first component.

A First Component
=================

Let us create a React component that displays today's date. We will use [create-react-app](https://github.com/facebookincubator/create-react-app) because it is a good starting point and a defacto standard.

First, install create-react-app if you don't have it already:

```
> npm install -g create-react-app
```

Then scaffold a new application:

```
> create-react-app a-first-component
> cd a-first-component
> npm start
```

You will see the default application scaffold rendered in your browser. 

Open the file `src/index.js`. It should look like:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(<App />, document.getElementById('root'));
registerServiceWorker();
```

A React component is just a function that returns some HTML (technically JSX which is a HTML-like markup language that is compiled to JavaScript). A React component that shows the current date is:

```javascript
function Today() {
    return <div>Today is {new Date().toString()}</div>;
}
```

Putting that into `src/index.js` and rendering it instead of `<App />` we now have `src/index.js` containing:

```javascript
// ... imports as above

function Today() {
    return <div>Today is {new Date().toString()}</div>;
}

ReactDOM.render(<Today />, document.getElementById('root'));
```

And the browser renders our app as:

```
Today is Wed Jan 03 2018 15:16:43 GMT+1000 (AEST)
```

Summary
======

Creating a React app with the most basic component I can think of is simple. In future posts I will build on this on step at a time until we have a viable application.

> Next: [Learning React 2 - Data in with props](/articles/2018-01-03-react-2-props/)