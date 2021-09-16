---
title: Implementing Tetris with React and Redux - VI
author: Liam McLennan
date: 2016-01-07 20:32
template: article.jade
---

Last time I promised you collapsing completed rows and ending the game. That will have to wait. I have something **even more exciting!**.

> Converting to typescript

To follow along with this article checkout the commit [70fa71](https://github.com/liammclennan/tetris/commit/70fa71e6893c73dff57ae29ff006ab51f3896136).

```
$ git clone https://github.com/liammclennan/tetris.git
$ git checkout 70fa71
$ npm run bootstrap
$ npm run likehell
```

Typescript
==========

Having a type checking compiler eliminates a large set of bugs, and makes a heap of tests redundant. Tests are a debt that must be serviced so we should aim to get maximum value from the fewest tests.

[Typescript](http://www.typescriptlang.org/) adds compile time type checking to JavaScript. Converting our application to typescript requires:

* converting the application code from JavaScript to Typescript
* acquiring [Typescript type definitions](http://www.typescriptlang.org/Handbook#writing-dts-files) for all dependencies
* modifying the build process to include the Typescript compiler

Adding Typescript to the Build
======

Working backwards we can start by acquiring the required Typescript dev tools:

    npm install typescript tsd --save

Typescript is the npm package including the typescript compiler. [Tsd](http://definitelytyped.org/tsd/) is a tool for acquiring Typescript type definition files for popular JavaScript libraries.

We can then use tsd to fetch the type definitions we need:

```
tsd install react-global mousetrap mocha node underscore --save
```

The type definitions are saved into a *typings/* directory and listed in a manifest *tsd.json*. A Typescript file *typings/tsd.d.ts* references all of the type definitions as a shortcut way to import them all. I could not find a tsd definition for Redux so I manually included the type definition at *src/redux.d.ts*.

Next we prepend another build command in *gulpfile.js*.

    tsc src/app.tsx test/modelsTests.ts --jsx react --outDir build/es6 -t ES6

`tsc` is the typescript compiler.

`src/app.tsx` and `test/modelsTests.ts` are the entry point files. All other files are found by following dependencies. The `.tsx` extension indicates that the file contains [jsx](https://facebook.github.io/jsx/) embedded in Typescript.

`--jsx react` tells the compiler that when it finds jsx it should compile it to React JavaScript.

`--outDir build/es6` specifies where the output should be written.

`-t ES6` sets the compiler to target ES6 (ES2015).

Converting to Typescript
=======================

The easiest way to convert an application from JavaScript to Typescript is to start from the leaves of the dependency tree and work back to the root. That means starting with the modules that have the fewest dependencies. For this Tetris app *model.js* is the best choice.

Firstly we can change the file extension to *.ts* and attempt to compile using:

    tsc src/model.js -t ES6

which won't work. The typescript compiler will give messages indicating the things that need to be fixed. For *model.js* this is mostly the syntax of classes. Typescript has a shortcut syntax for specifying the properties of classes that looks like:

```
export class Tetromino {
  constructor(public name:string, public rotator: (rotation:string)=>Point[]) {}
}
```

This automatically creates public properties `name` and `rotator`. `name` has the type `string`. `rotator` has the type `(rotation:string)=>Point[]` which means a function from a `string` to an array of `Point`.

When all the type errors in *model.ts* are fixed we move to *components.ts*. This file is slightly more complicated because it has dependencies. The first thing to do is add a reference to *tsd.d.ts*.

    /// <reference path="../typings/tsd.d.ts" />

This is a directive to the typescript compiler that tells it where to find type information. From *tsd.d.ts* typescript can find type definitions for all of the libraries we are using (except Redux).

The big change in this file is specifying the `props` and `state` type parameters for each React component by passing the types to the `createClass` function.

For the `GameView` component the type of its props is:

```
interface GameViewProps {
  game: Model.Game;
}
```

`GameView` does not use state (none of our components do) so the component changes to:

```
export var GameView = React.createClass<GameViewProps,any>({
  render: function () {
    return <div className="border" style={
      {
        width: this.props.game.cols*25,
        height: this.props.game.rows*25
      }}>
      <PieceView piece={this.props.game.fallingPiece} />
      <RubbleView rubble={this.props.game.rubble} />
    </div>;
  }
});
```

We now move up the dependency tree to *app.tsx* which needs to reference *tsd.d.ts* and *redux.d.ts*:

    /// <reference path="../typings/tsd.d.ts" />
    /// <reference path="redux.d.ts" />

The file is otherwise much the same.

The test file *modelsTests.ts* also needs a reference to *tsd.d.ts*. Then everything should compile properly, the tests should run successfully and the application should run.

And now we have type safety.
