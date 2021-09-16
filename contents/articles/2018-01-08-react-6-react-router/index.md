---
title: Learning React 6 - React Router
author: Liam McLennan
date: 2018-01-08 20:32
template: article.jade
---

[Learning React 5 - React-Redux](/articles/2018-01-05-react-5-react-redux/) explored the helpers provided by Redux for integration into a React application. Our today application is now using Redux to store and manage state. 

Applications designed for the web should be URL oriented. Adhering to this expectations enables navigation, caching, book marking and many other standard activities. Implmenting routing in a React application requires a routing library, and the most popular is [React Router](https://reacttraining.com/react-router/web/guides/quick-start).

React Router
===========

In this post we will add routing, using React Router, to our today application. We will support URLs like,

```
localhost:3000/day/2018-03-20
```

That will set the `day` value to March 20. Instead of having to use the 'Next' button users will be able to navigate directly to day of their choosing, and instead of the 'Next' button being implemented as a state change it will become a simple navigation link to the next day. 

To start, as always, install the new library:

```
> npm install react-router-dom --save
```

To add the router, without modifying the existing behaviour we: 

* import some React Router functions
* add a `Router` to our component tree
* add a `Route` component that maps a URL to a component

```javascript
import { BrowserRouter as Router, Route } from 'react-router-dom';

ReactDOM.render(
    <Provider store={store}>
        <Router>
            <Route exact path="/" component={ConnectedToday} />
        </Router>
    </Provider>, 
    document.getElementById('root'));
```

This declares that the root path ('/') should result in the the `ConnectedToday` component being rendered in the position specified. 

Now our application is using a router, but behaving exactly as before. 

Let's add another route for **/day/YYYY-MM-DD**:

```html
ReactDOM.render(
    <Provider store={store}>
        <Router>
            <div>
                <Route exact path="/" component={ConnectedToday} />
                <Route path="/day/:datestring" component={ConnectedToday} />
            </div>
        </Router>
    </Provider>,
    document.getElementById('root'));
```

Note that `:datestring` represents a querystring parameter variable. Next, modify the `Today` component to read its date from the router instead of from Redux:

```javascript
function Today(props) {
    const day = new Date(props.match.params.datestring);
    return <div>Today is {day.toString()} 
        <button type="button" onClick={props.onNext}>Next</button></div>;
}
```

React Router uses [the same trick](https://reactjs.org/docs/context.html) as Redux to supply route parameters to components without having to manually pass them as props. 

Now you can navigate to a URL like *http://localhost:3000/day/1994-5-8* to get the Today component rendered for the 8th May 1994. We have broken the next button because it increments a `day` value in the Redux store, but our component takes its day value from the router. Instead of incrementing a value in the store we can instead use a link to the next day:

```javascript
function Today(props) {
    const day = new Date(props.match.params.datestring);
    const nextDay = new Date(day.getTime() + (24*3600*1000));
    const tomorrowDateString = `${nextDay.getFullYear()}-${nextDay.getMonth()+1}-${nextDay.getDate()}`;
    return (
        <div>Today is {day.toString()} 
            <Link to={"/day/" + tomorrowDateString}>Next</Link>
        </div>);
}
```

`Link` is a React Router component used to generate links.

Now our UI is bound to the URL. As we navigate through consecutive days you will see the URL change. Also, if you manually change the URL the UI is updated to match. To make our routing more interesting, let's add a `Home` component on the root URL:

```html
ReactDOM.render(
    <Provider store={store}>
        <Router>
            <div>
                <Route exact path="/" component={Home} />
                <Route path="/day/:datestring" component={ConnectedToday} />
            </div>
        </Router>
    </Provider>,
    document.getElementById('root'));
```

The `Home` component should display a link to a randomly selected day:

```javascript
function Home() {
    const randomBetween = (min, max) => Math.floor(Math.random() * (max-min+1) + min);
    const randomDateString = 
        `${randomBetween(1900, 2100)}-${randomBetween(1, 12)}-${randomBetween(1, 28)}`;

    return <div><h1>Home</h1>
        <Link to={`/day/${randomDateString}`}>{randomDateString}</Link>
    </div>;
}
```

With this completed we have a home component that links to a random day, that has a next button that advances (via the URL) to the next day. For this particular application at this time we could delete all of the Redux and Redux-router code, because we are no longer managing any state in Redux. 

redux-first-router
========

I chose Redux Router for this example because it is by far the most popular React router, but the awkwardness of integrating React Router and Redux into an application is bothersome. An alternative is to use a [redux-first-router](https://github.com/faceyspacey/redux-first-router) instead of React Router. redux-first-router does not bind routes to components, instead it binds routes to actions. When a route is matched redux-first-router dispatches an action to that effect and everything else is handled the Redux way with reducers. 

Summary
======

URLs are the central abstraction of a well-behaved web application. To achieve this with React we need a router. React router is the popular choice although it does have some friction with state management solutions like Redux. redux-first-router is another alternative. 

> Next: [Learning React 7 - AJAX](/articles/2018-01-10-react-7-ajax/)

Get The Code
------------

The code for this example is [on Github](https://github.com/liammclennan/today-react-app). You can access the code as it was at the completion of this step by cloning the repository and checking out the tag that corresponds to this post. 

```bash
git clone https://github.com/liammclennan/today-react-app.git
git checkout react6 
```

or browse at https://github.com/liammclennan/today-react-app/tree/4f53d611e3ecaa3d5bcfe92e970e56fcf6981860.