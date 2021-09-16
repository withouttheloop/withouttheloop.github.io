---
title: Learning React 9 - Data Bound Routes
author: Liam McLennan
date: 2018-01-11 20:32
template: article.jade
---

[Learning React 8 - AJAX Continued](/articles/2018-01-10-react-8-ajax-continued/) has our application capable for searching a movie API and displaying the list of results. Now we would like to be able to view the details for a selected movie. 

A good web application should have a URL for every addressable state, so selecting a movie should change the URL. We will use, `/movie/:imdbID`. First add an action to the `routesMap`:

```javascript
const routesMap = { 
    HOME: '/',
    MOVIE: '/movie/:imdbID'
};
```

Then create a new movie component:

```javascript
function Movie(props) {
    return <div>
            { props.movie 
                ? <p>
                        <h1>{props.movie.Title}</h1>
                        <img src={props.movie.Poster} alt={`${props.movie.Title} poster`} />
                    </p>
                : <p>Loading...</p>
            }
            </div>;
}

const ConnectedMovie = connect(
    function mapStateToProps(state) {
        return state.movie;
    }, 
    function mapDispatchToProps(dispatch) {
        return {};
    }
)(Movie);
```

and update the `App` component to display the `ConnectedMovie` component when the `MOVIE` action is dispatched. 

```javascript
const componentMap = {
    HOME: ConnectedSearch,
    MOVIE: ConnectedMovie
};
```

*redux-first-router* Route Thunks
========

Now we encounter a problem. When the router sees the route `/movie/:imdbID` it will cause the application to show the `ConnectedMovie` component, which expects to display a movie using data from the redux store, but the data is not there. Somehow we need to take the `imdbID` from the route and populate the redux store. *redux-first-router* allows for this using an alternative way of specifying the action-to-route mapping, with an included `thunk` value. This is a function that runs when the route is matched and that has access to the current store. Within the `thunk` we can read the `imdbID` from the store (*redux-first-router* put it on the `location` key for us), perform a request, and publish the `MOVIE_FETCHED` action with the data when we are done. 

```javascript
const routesMap = { 
    HOME: '/',
    MOVIE: {
        path: '/movie/:imdbID',
        thunk: async (dispatch, getState) => {
            dispatch({ type: "MOVIE_CLEAR"});
            const imdbID = getState().location.payload.imdbID;
            const movie = await fetch(`http://www.omdbapi.com/?apikey=8e4dcdac&i=${imdbID}`)
            .then((response) => response.json());
            dispatch({ type: "MOVIE_FETCHED", movie });
        }
    }
};
```

The `MOVIE_CLEAR` action is dispatched before the request to clear the previous movie data from the store. The `movieReducer` then copies the data from the action into the store.

```javascript
const defaultState = {movie: null};

export const movieReducer = {
    movie: function (state = defaultState, action) {
        switch (action.type) {
            case "MOVIE_CLEAR": 
                return Object.assign({}, state, defaultState);
            case "MOVIE_FETCHED":
                return Object.assign({}, state, {movie: action.movie});
            default: return state;
        }
    }
};
```

This is what I mean by a 'data bound route'. Whenever the route `/movie/:imdbID` is seen we fetch the data for that route and put it in the store. 

If an AJAX request is required because of a user action then dispatch an action, with a promise `payload`, in the `mapDispatchToProps` function. *redux-promise-middleware* will resolve the promise and publish the `_FULFILLED` version of the action, which can be processed in a reducer. 

If a route requires an AJAX request we need a different strategy, because routes map to actions, which are processed in reducers, which don't have the ability to dispatch new actions. *redux-first-router* provides the `thunk` option so that we have a place to do asynchronous operations in response to URL changes, and publish actions when we have a result.

Converting Event Actions to Route Actions
=========================================

Above I said,

> A good web application should have a URL for every addressable state

yet, when we perform a search, the application state changes but the URL does not. When you think about it, most actions represent a state change that can, and should, be reflected in a URL change. When I search for 'Terminator' I want the url to change to `/search/Terminator`. Thus a search for terminator becomes an addressable state that I can link directly to.

*redux-first-router* synchonizes in both directions. If the URL changes it publishes the corresponding route action. If a route action is dispatched it changes the URL. To make search results addressable we will handle the search form submission, dispatch a route action, then do the search using the route handling thunk. First, modify the action that is dispatched when a user searches:

```javascript
function mapDispatchToProps(dispatch) {
    return {
        onSearch: (title)=> {
            dispatch({
                type: 'SEARCH',
                payload: {title}
                });
        } 
    };
}
```

This `SEARCH` action will be a route action. Next, add the `SEARCH` action to the route configuration, mapped to `/search/:title`. When the route is matched, read the `title` from the Redux store, query the API, and then dispatch the `SEARCH_FULFILLED` action that the `searchReducer` is already expecting. 

```javascript
const routesMap = { 
    // ...
    SEARCH: {
        path: '/search/:title',
        thunk: async (dispatch, getState) => {
            const title = getState().location.payload.title;
            const results = await fetch(`http://www.omdbapi.com/?apikey=8e4dcdac&s=${encodeURIComponent(title)}`)
                .then((response) => response.json())
            dispatch({ type: "SEARCH_FULFILLED", payload: results });
        }
};
```

The routing for the application is now:

| Route action  | URL           | Thunk | Component  |
| ------------- |---------------| -----|
| `HOME`      | `/` | | ConnectedSearch |
| `MOVIE`  | `/movie/:imdbID` | Publishes `MOVIE_CLEAR` actions. Fetches movie data by `imdbID`. Publishes `MOVIE_FETCHED` action. | ConnectedMovie |
| `SEARCH` | `/search/:title` | Fetches results by `title`. Publishes `SEARCH_FULFILLED`. | ConnectedSearch |

&nbsp;

As the `onSearch` handler no longer publishes an action with a promise we are no longer using *redux-promise-middleware*. Many actions can be route changes and do their asynchronous operations with *redux-first-router* thunks. Actions that do not make sense as route changes, but that still require an asynchronous operation, should still use *redux-promise-middleware*.

Summary
======

Where sensible, states should be addressable via a route. Where a route requires an asynchronous operations, such as making an AJAX request, use the `thunk` property of the route configuration object to do the asynchronous operation and dispatch an action when it is complete.

> Next: [Learning React 10 - Typescript](/articles/2018-01-17-react-10-typescript/)

Get The Code
------------

The code for this example is [on Github](https://github.com/liammclennan/movie-library). You can access the code as it was at the completion of this step by cloning the repository and checking out the tag that corresponds to this post. 

```bash
git clone https://github.com/liammclennan/movie-library.git
git checkout react9
```

or browse at https://github.com/liammclennan/movie-library/tree/react9.

