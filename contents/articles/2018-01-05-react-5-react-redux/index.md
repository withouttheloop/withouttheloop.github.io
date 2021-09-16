---
title: Learning React 5 - React-Redux
author: Liam McLennan
date: 2018-01-05 20:32
template: article.jade
---

In [Learning React 4 - Redux](/articles/2018-01-05-react-4-redux/) we learnt about what Redux can do for your React 
application and how it can be integrated. The result was the following code:

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

Note that the `Today` component is not coupled to the Redux store (which is good), but also that we are manually passing data into and out of the `Today` component. This is fine for our example but typically a component is contained many levels deep in a component hierarchy. The design we have so far would require us to pass the `day` prop manually down through each level of the hierarchy and the `onNext` prop manually back up through each level of the hierarchy. 

React-redux
==========

React, and Redux, provide a solution to this problem, implemented in a package called [React-redux](https://github.com/reactjs/react-redux). Install in the usual way:

```
> npm install react-redux --save
```

React-redux provides a way to connect a component's props to the pieces of the Redux state that the component cares about. When creating a component you get this work by:

* defining the mapping from the Redux state to the components `props`
* defining the mapping from the components props to actions dispatched to the Redux store

The above mappings are combined with an existing component to produce a new component, that is connected to Redux. For the `Today` component this is:

```javascript
import { connect, Provider } from 'react-redux';

function Today({ day, onNext }) {
    return <div>Today is {day.toString()}
        <button type="button" onClick={onNext}>Next</button></div>;
}

const ConnectedToday = connect(
    // Today uses the 'day' property of the Redux state. 
    // Which is the whole thing. 
    function mapStateToProps(state) {
        return state;
    }, 
    function mapDispatchToProps(dispatch) {
        return {
            // This 'onNext' function is provided as a prop to the 'Today' component, 
            // but with 'dispatch' in scope. 
            onNext: ()=> { dispatch({type: "NEXT_DAY"}); }
        };
    }
    )(Today);
```

`ConnectedToday` is a new component that wraps the `Today` component and connects it to the Redux store. Now when we `render` our application we can use the `ConnectedToday` component instead of the `Today` component, and remove the props. 

To make it work we must wrap our component tree in a `Provider`, that makes the store available to connected components. We can also drop the subscription to the store (React-redux takes care of this for us):

```javascript
ReactDOM.render(
    <Provider store={store}>
        <ConnectedToday />
    </Provider>, 
    document.getElementById('root'));
```

Now we have achieved our goal. The today component is still independent of the Redux store, and we have avoided manually passing state and events up and down the component hierarchy. Here is the final code:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import registerServiceWorker from './registerServiceWorker';
import { createStore } from 'redux';
import { connect, Provider } from 'react-redux';

function Today({ day, onNext }) {
    return <div>Today is {day.toString()} 
        <button type="button" onClick={onNext}>Next</button></div>;
}

const ConnectedToday = connect(
    function mapStateToProps(state) {
        return state;
    }, 
    function mapDispatchToProps(dispatch) {
        return {
            onNext: ()=> { dispatch({type: "NEXT_DAY"}); }
        };
    }
    )(Today);

function reducer(state = { day: new Date(2018,2,20) }, action) {
    switch (action.type) {
        case "NEXT_DAY":
            const currentDay = store.getState().day;
            return { day: new Date(currentDay.getTime() + (24*3600*1000))};
        default: return state;
    }
}
const store = createStore(reducer);

ReactDOM.render(
    <Provider store={store}>
        <ConnectedToday />
    </Provider>, 
    document.getElementById('root'));

store.dispatch({type: "START"});

registerServiceWorker();
```

Summary
======

When React and Redux are used together 'React-redux' is usually used as a convenient way to join them together. It is simply a matter of:

* Wrapping the component tree in a `<Provider/>`
* Providing a `mapStateToProps` and `mapDispatchToProps` function for each connected component

> Next: [Learning React 6 - React Router](/articles/2018-01-08-react-6-react-router/)

Get The Code
------------

The code for this example is [on Github](https://github.com/liammclennan/today-react-app). You can access the code as it was at the completion of this step by cloning the repository and checking out the tag that corresponds to this post. 

```bash
git clone https://github.com/liammclennan/today-react-app.git
git checkout react-5 
```

or browse at https://github.com/liammclennan/today-react-app/tree/18a03e73197b15eef0746a01fdd111942ff1a2d3.