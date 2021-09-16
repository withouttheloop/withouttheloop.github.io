---
title: Implementing Tetris with React and Redux - II
author: Liam McLennan
date: 2015-12-27 20:32
template: article.jade
---

In the previous article we began implementing tetris with React and Redux and got as far rendering the tetris shapes (called tetrominos).

To follow along with this article checkout the commit [594a735](https://github.com/liammclennan/tetris/commit/594a7355c377f62edb44a1cb3f10759d0e0bf804).

```
$ git clone https://github.com/liammclennan/tetris.git
$ git checkout 594a735
```

Simplifying the Components
=======

Thinking ahead it becomes apparent that we will need logic related to the shapes, such as how to rotate them and prevent collisions. If we introduce models for the tetrominos then they will be represented twice, once as models and once as React components.

As we don't want to repeat ourselves I think it is appropriate to simplify the React components. Instead of having different components for each tetromino all we really need is one component that can render a square. At the same time I will begin to introduce ES2015 modules. My new module, called components, will be in a file called `components.js`. Anything that I want to be available outside of the module needs to be prefixed with the `export` keyword. Instead of loading React using the commonjs `require` function we will switch to using the `import` keyword from ES2015. The new components module, and the new `ShapeView` component looks like:

```
import * as React from 'react';
import * as ReactDOM from 'react-dom';

export var ShapeView = React.createClass({
  render: function () {
    return <div>
      {this.props.shape.squares().map(sq => <Square row={sq.row} col={sq.col} />)}
    </div>;
  }
});
```

The `import * as React from 'react'` statement means that we are importing the `react` module and assigning all of its exports to the identifier `React`.

The refactored `ShapeView` component expects a model called `shape` having a method `squares()` that returns a collection of points that defines the shape. Thus the component can be reused for any shape that can describes its points.

Introducing Models
=================

Having recognised that we need models to encapsulate the Tetris logic I will create another module, called `models`. The file models.js contains classes that model some tetrominos (O and L) and a class that models the Squares from which the other shapes are composed.

```
export class Sqr {
  constructor(row,col) {
    this.row = row;
    this.col = col;
  }
}

export class O {
  constructor(row,col) {
    this.row = row;
    this.col = col;
  }
  squares() {
    return [new Sqr(this.row,this.col), new Sqr(this.row, this.col+1), new Sqr(this.row+1,this.col), new Sqr(this.row+1,this.col+1)];
  }
}

export class L {
  constructor(row,col) {
    this.row = row;
    this.col = col;
  }
  squares() {
    return [new Sqr(this.row,this.col), new Sqr(this.row+1, this.col), new Sqr(this.row+2,this.col), new Sqr(this.row+3,this.col)];
  }
}
```

As required by the `ShapeView` component the `O` and `L` classes both have a `squares()` method returning an array of `Sqr`.  

Putting it Together
================

Now we can simplify out `app.js`. Firstly, import the required modules so that they are available within `app.js`:

```
var React = require('react');
var ReactDOM = require('react-dom');
var Components = require('./components');
var Model = require('./model');
```

Note that I have not yet switched this file to use the ES2015 style imports.

Next, I create an object to act as the model for React to render:

    var data = [new Model.O(1,1), new Model.L(1,4)];

For now, the model is an array of tetrominos, one `O` and one `L`.

The rendering is now simpler. All we need to do is loop over the total set of squares and render each.

```
ReactDOM.render(
<div>
  {data.map(c => <Components.ShapeView shape={c} />)}
</div>,
    document.getElementById('container')
);
```

In the next article we will start animating the tetrominos and introduce `Redux` as a tool for managing state.
