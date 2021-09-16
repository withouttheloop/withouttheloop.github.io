---
title: Learning React 4 - Redux
author: Liam McLennan
date: 2018-01-05 20:32
template: article.jade
---

The post [Learning React 3 - Data out with props](/articles/2018-01-03-react-3-data-out/) finished with a 
component that is customised by data passed in, and that publishes user events, which modifies that application state, and causes the component to update. 

```javascript
function Today({ day, onNext }) {
    return <div>
                Today is {day.toString()}
                <button type="button" onClick={onNext}>Next</button>
            </div>;
}

let day = new Date(2018,2,20);

function onNext() {
    day = new Date(day.getTime() + (24*3600*1000)); 
    render();
}

function render() {
    ReactDOM.render(<Today day={day} onNext={onNext} />, 
        document.getElementById('root'));
}
render();
```

To keep the `Today` component simple and composable the state has been entirely pulled out of the component (into the `day` variable).

As applications become more complex it becomes more difficult to work with this external state and keep track of the modifications that need to be made in response to events. 

Redux
====

[Redux](https://redux.js.org/docs/introduction/) is a tool for managing application state and controlling changes to that state. I find the Redux documentation unnecessarily confusing. All you really need to know to understand Redux is:

* Redux holds state in a single Javascript object, called a store.
* Redux uses functions (sometimes referred to as reducers) to apply events (sometimes called actions) to update the application state. 
* Redux has a function `createStore` that creates a store.
* Redux has a function `getState` that gets the application state from the store.
* Redux has a function `subscribe` that registers listeners to be called when the application state changes. 
* Redux has a function `dispatch` that is used to send an event (action) to the reducer to update the store.

Consider our Today component. First we add redux to our project:

```
> npm install redux --save
```

Now we can put our application state into a redux store:

```javascript 
function reducer(state = { day: new Date(2018,2,20) }, action) {
    return state;
}
const store = createStore(reducer);

store.subscribe(() => {
    ReactDOM.render(<Today day={store.getState().day} onNext={onNext} />, 
        document.getElementById('root'));
});

store.dispatch({type: "START"});
```

That is enough to get the application to render with its initial state. Note that the initial application state (`{ day: new Date(2018,2,20) }`) is set as the default value of the `state` argument. 

The event handling is broken because we can no longer directly modify the `day` variable. Instead we must dispatch an appropriate action (`NEXT_DAY`) to the redux store. In the reducer function, if we see our action (`NEXT_DAY`) then we return a new application state with the `day` value advanced by one day:

```javascript 
function onNext() {
    store.dispatch({type: "NEXT_DAY"});
}

function reducer(state = { day: new Date(2018,2,20) }, action) {
    switch (action.type) {
        case "NEXT_DAY":
            const currentDay = store.getState().day;
            return { day: new Date(currentDay.getTime() + (24*3600*1000))};
        default: return state;
    }
}
```

Both the state object and the action objects can be any Javascript value. It is a convention to include a `type` property on action values. The `type` property can then be used in reducer functions to determine which event/action has occurred so that the application state can be updated appropriately. 

The full program is now:

```javascript
function Today({ day, onNext }) {
    return <div>Today is {day.toString()}
        <button type="button" onClick={onNext}>Next</button></div>;
}

function onNext() {
    store.dispatch({type: "NEXT_DAY"});
}

function reducer(state = { day: new Date(2018,2,20) }, action) {
    switch (action.type) {
        case "NEXT_DAY":
            const currentDay = store.getState().day;
            return { day: new Date(currentDay.getTime() + (24*3600*1000))};
        default: return state;
    }
}
const store = createStore(reducer);

store.subscribe(() => {
    ReactDOM.render(<Today day={store.getState().day} onNext={onNext} />, 
        document.getElementById('root'));
});

store.dispatch({type: "START"});
```

Summary
=======

Redux can be confusing, because of all the additional machinery that has accumulated around it, and because of the way the documentation is presented. At its core Redux is just a container for application state and a formalized way of modifying that application state.

Good React components are simple, pure functions of their inputs (props). Redux is a helpful tool that makes this much easier.

> Next: [Learning React 5 - React-redux](/articles/2018-01-05-react-5-react-redux/)

Get The Code
------------

The code for this example is [on Github](https://github.com/liammclennan/today-react-app). You can access the code as it was at the completion of this step by cloning the repository and checking out the tag that corresponds to this post. 

```bash
git clone https://github.com/liammclennan/today-react-app.git
git checkout react4 
```

or browse at https://github.com/liammclennan/today-react-app/tree/9f17db0e8d19c505efaaa7e828cb71e535e2377a.