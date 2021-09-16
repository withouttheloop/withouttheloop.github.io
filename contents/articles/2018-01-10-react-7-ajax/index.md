---
title: Learning React 7 - AJAX
author: Liam McLennan
date: 2018-01-10 20:32
template: article.jade
---

In [Learning React 6 - React Router](/articles/2018-01-08-react-6-react-router/) we learned how to connect a client-side router to a React application. 

In this post we will look at how to add AJAX HTTP requests to a React application. This requires more thought than introducing AJAX to products built with other libraries, because our React application so far has used pure components (components that produce output entirely determined by their inputs) and been entirely synchronous (processing proceeds continuously without having to wait for any external events).

Movie Library
=====

For this demonstration we will need a more complicated application than the one we have used so far, so we will build a movie library. The application will allow the user to:

* search the [OMDB movie database API](http://www.omdbapi.com) for movies.
* view the details of a movie 
* select movies to add to their favourite list
* view their favourite list

Start by creating a new application with create-react-app:

```bash
$ create-react-app movie-library
```

and start the app:

```bash
$ npm start
```

Install redux, react-redux and a router ([redux-first-router](https://github.com/faceyspacey/redux-first-router)):

```bash
$ npm install --save redux react-redux history redux-first-router redux-first-router-link
```

Modify `index.js` to use a default redux implementation:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import registerServiceWorker from './registerServiceWorker';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import App from './App';

function reducer(state = { }, action) {
    switch (action.type) {
        default: return state;
    }
}
const store = createStore(reducer);

ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>, 
    document.getElementById('root'));

store.dispatch({type: "START"});

registerServiceWorker();
```

The Search Component
====================

The starting page of the movie library will feature a movie search component. 

Start by creating a new file, `Search.js` containing a minimal Search component:

```javascript
import React from 'react';
import { connect } from 'react-redux';

function Search() {
    return <div>
        <h1>Search</h1>
    </div>;
}

export const ConnectedSearch = connect(
    function mapStateToProps(state) {
        return state;
    }, 
    function mapDispatchToProps(dispatch) {
        return {};
    }
)(Search);
```

The `Search` module exports a component `ConnectedSearch` that is our search component wrapped in a react-redux connector to give it access to the redux store (we will need this later). Now, modify `App.js` to import and use the `ConnectedSearch` component:

```javascript
 import React, { Component } from 'react';
import './App.css';
import { ConnectedSearch } from './Search';

class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <h1 className="App-title">Movie Library</h1>
        </header>
        <ConnectedSearch />
      </div>
    );
  }
}

export default App;
```

Now the application should be working a displaying a heading 'Search'.

The `Search` component needs a text input. We will create a new component `SearchForm` to contain the text input. To get the form to function properly we must use React's component 'state' feature, which means using the [class component syntax](https://reactjs.org/docs/state-and-lifecycle.html) instead of the function syntax. Both 'state' and the class syntax are undesirable, hence why they are contained in the smallest possible area where they are absolutely needed (the `SearchForm` component).

```javascript
class SearchForm extends React.Component {
    constructor(props) {
        super(props);
        this.state = {title: ""}
        this.titleChange = this.titleChange.bind(this);
        this.search = this.search.bind(this);
    }

    titleChange(event) {
        this.setState({title: event.target.value});
    }

    search(event) {
        event.preventDefault();
        this.props.onSearch(this.state.title);
    }

    render() {
        return <form onSubmit={this.search}>
            <label htmlFor="title">Title: </label>
            <input type="text" name="title" value={this.state.title} onChange={this.titleChange}/>
        </form>;
    }
}

function Search() {
    return <div>
        <h1>Search</h1>
        <SearchForm />
    </div>;
}

export const ConnectedSearch = connect(
    function mapStateToProps(state) {
        return state;
    }, 
    function mapDispatchToProps(dispatch) {
        return {
            onSearch: (title)=> { dispatch({type: "SEARCH", title}); }
        };
    }
)(Search);
```

When a user types a character in the textbox it fires the `onChange` event, which [updates the components state to have the latest value of the textbox](https://reactjs.org/docs/forms.html), which causes the component to re-render and display the character the user typed in the textbox. 

When the form is submitted we call the `onSearch` function on `props`. The React-Redux connection translates this into a dispatch of an action with `type: "SEARCH"`.

Summary
=======

We started building a React movie library with React, Redux, react redux, and redux-first-router. We implemented a class component for a form, with local state and connected everything to our Redux store. 

> Next: [Learning React 8 - AJAX Continued](/articles/2018-01-10-react-8-ajax-continued/)

Get The Code
------------

The code for this example is [on Github](https://github.com/liammclennan/movie-library). You can access the code as it was at the completion of this step by cloning the repository and checking out the tag that corresponds to this post. 

```bash
git clone https://github.com/liammclennan/movie-library.git
git checkout react7 
```

or browse at https://github.com/liammclennan/movie-library/tree/7548d450b7727f124070dd7ee9b73575d740e9b3.