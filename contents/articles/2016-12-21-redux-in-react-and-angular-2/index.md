---
title: Redux in React and Angular 2
author: Liam McLennan
date: 2016-12-21 20:32
template: article.jade
---

Earlier this month I presented at [dddbrisbane](http://dddbrisbane.com/) about simplifying UI programming and I included the simplest React + Redux application I could come up with.

The simplest React + Redux application I could come up with
===============

It displays a random number. Clicking the text generates a new random number. Toggle the `Result` button to see the code more easily. 

<p data-height="265" data-theme-id="0" data-slug-hash="xRrzXO" data-default-tab="js,result" data-user="liammclennan" data-embed-version="2" data-pen-title="FknRooowwwmmm" class="codepen">See the Pen <a href="http://codepen.io/liammclennan/pen/xRrzXO/">FknRooowwwmmm</a> by Liam McLennan (<a href="http://codepen.io/liammclennan">@liammclennan</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

It uses the [React function component syntax](https://facebook.github.io/react/docs/components-and-props.html) to define the component as a function from UI state (`props`) to HTML (a H1 tag).

```js
function HelloNumber(props) {
  function handleClick(e) {
    store.dispatch({type:'MAKE_A_NEW_NUMBER'});
  }
  return <h1 onClick={handleClick}>A random number {props.number}</h1>;
}
```

The `onClick` handler dispatches a `MAKE_A_NEW_NUMBER` action to the redux store. 

The UI state for the application is an object with a property `number`:

```js
{ number: Math.random() }
```

The [redux reducer](http://redux.js.org/docs/basics/Reducers.html) (function responsibile for applying actions to the state to make a new state) handles the `MAKE_A_NEW_NUMBER` number by setting it to a new random number. 

```js
let store = Redux.createStore(function (state = { number: Math.random() }, action) {
  switch (action.type) {
    case 'MAKE_A_NEW_NUMBER':
      return Object.assign({},state, {number:Math.random()});
    default: return state;
  }
});
```

Now dispatching an application has updated the state, but we still need to have state changes trigger the UI to update, which is done via the `subscribe` method:

```js
store.subscribe(()=>{
  ReactDOM.render(
    <HelloNumber number={store.getState().number}  />, 
    document.getElementById('container')
  );
});
```

and finally prime the whole thing by an initial dispatch

```js 
store.dispatch({type:'MAKE_A_NEW_NUMBER'});
```

This is not strictly necessary but it is an easy way to trigger the `subscribe` callback and cause the initial render. 

The simplest Angular 2 + Redux application I could come up with
===============

What would the equivalent Angular 2 application look like?

I couldn't find a simple way to develop an Angular 2 application in Codepen or similar so instead I worked locally, starting with the [Angular 2 Quickstart](https://angular.io/docs/ts/latest/quickstart.html). 

To reproduce the random number display functionality in Angular 2 I found that I needed a component hierarchy (parent + child). 

The child component contains the header and dispatches a `NEW_RANDOM_NUMBER` action when clicked, just like the React component. Two values are passed into the child component, from the parent, via `@Input()` decorators: the application state (`appState`) and the redux store dispatch function (`dispatch`).  

```js
import { Component,Input } from '@angular/core';

@Component({
  selector: 'my-child',
  template: `<h1 (click)="onClickMe()">A random number {{appState.number}}</h1>`,
})
export class ChildComponent  { 
    @Input() appState:any;
    @Input() dispatch: (action:any) => void;

    onClickMe() {
        this.dispatch({type:'NEW_RANDOM_NUMBER'});
    }
 }
```

The parent component (`AppComponent`) creates  the redux store, using the same reducer as the React version. The UI state is copied to a local field (`appState`) and updated whenever the store's state changes. The `appState` and redux `dispatch` function are bound to the child component (`<my-child/>`). 


```js
import { Component,Input } from '@angular/core';
import { ChildComponent } from './child.component';
import { createStore } from 'redux'

interface HasANumber {
  number: number;
}
interface Action {
  type: string;
}

@Component({
  selector: 'my-app',
  template: `<my-child [appState]="appState" [dispatch]="store.dispatch"></my-child>`
})
export class AppComponent  { 
  store = createStore<HasANumber>(function (state = {number:Math.random()}, action:Action) {
    switch (action.type) {
      case "NEW_RANDOM_NUMBER": 
        return {number:Math.random()};
      default: return state;
    }
  });
  appState = this.store.getState();

  constructor() {
    this.store.subscribe(()=> {
      this.appState = this.store.getState();      
    });
  }
}
```

Final Thoughts
========

Having Typescript easily available in Angular 2 was nice. Further, I found that Angular 2 is at its best when imitating my very strict React style. Both frameworks work well with Redux and the uni-directional data flow idea. 