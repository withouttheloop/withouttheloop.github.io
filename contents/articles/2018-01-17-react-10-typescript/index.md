---
title: Learning React 10 - Typescript
author: Liam McLennan
date: 2018-01-17 20:32
template: article.jade
---

[Learning React 9 - Data Bound Routes](/articles/2018-01-11-react-9-data-bound-routes/) showed the *redux-first-router* way to implement data bound routes, that is, routes that require fetching some data to be able to render their view. This post will add static typing to our movie library application.

Types
=====

A type is a restriction on data. It defines the set of valid values for that type. E.g. all real numbers are valid for the type `number`, but only the whole numbers are valid for the type `integer`. All unicode characters are valid possibilities for the `character` type, but only 'true' and 'false' are valid values for the `boolean` type. 

Static Typing
=========

Static typing means that the types of a program are verified for correctness at compile time. The types flow through the program and everywhere they are used the compiler checks that the usage is valid for the type. A static type checker would prevent the following non-sensical operation:

```javascript
1 + 'a'
```

on the grounds that there is no good, semantically valid interpretation. It would also prevent `1 + 'a'.toNum()`, because the string type does not have a `toNum()` method. Thus a static type system detects a class of errors that would otherwise potentially occur at runtime.

Gary Bernhardt provided [an interesting overview](https://www.destroyallsoftware.com/compendium/types?share_key=baf6b67369843fa2).

To help us write correct React applications it is useful to add a static type checker. Two options are available: 

* [flow](https://flow.org/), developed by Facebook
* [typescript](https://www.typescriptlang.org/), developed by Microsoft

Both are great. This post will convert the movie library application to Typescript. 

Converting to Typescript
======

My preferred way to convert a *create-react-app* project to Typescript is by creating a new application and porting the code over. 

*[create-react-app-typescript](https://github.com/wmonk/create-react-app-typescript)* is a project that maintains a Typescript fork of *create-react-app*. To use it:

```bash
> create-react-app movie-library-ts --scripts-version=react-scripts-ts
```

This will create a new empty application in `movie-library-ts`. 

Next, add type definitions for the external libraries we are using:

```bash
> npm install @types/history @types/react @types/react-dom @types/redux @types/react-redux @types/redux-first-router @types/redux-first-router-link @types/redux-promise-middleware --save-dev
```

This allows Typescript to determine the types of the APIs exposed by those libraries. We use `--save-dev` because these dependencies are only required for compilation. They are not used at runtime. 

Add a file `Movie.tsx` to the `src/` directory. Typescript files have the `.ts` extension. Typescript files containing JSX have the `.tsx` extension. 

Typescript requires the `props` parameter to have a type, so declare a type `MovieData` having an optional property `movie` of type `MovieSummary` that contains the basic movie information. 

```javascript
interface MovieSummary {
    Title: string;
    Poster: string;
    imdbID: string;
}

interface MovieData {
    movie?: MovieSummary;
}

function Movie(props: MovieData)
    return (
        <div>
        { props.movie 
            ? <p>
                    <h1>{props.movie.Title}</h1>
                    <img src={props.movie.Poster} alt={`${props.movie.Title} poster`} />
                </p>
            : <p>Loading...</p>
        }
        </div>
    );
}
```

This guarantees that consumers of the movie component cannot pass the wrong props, or props of the wrong type. Next, add type information to the `connect` mappers:

```javascript
const ConnectedMovie = connect(
    function mapStateToProps(state: { movie: MovieData }) {
        return state.movie;
    }, 
    function mapDispatchToProps(dispatch: (action: any) => void) {
        return {};
    }
)(Movie);
```

Note the type of the `dispatch` function `(action: any) => void`. This declares that `dispatch` is a function with one parameter and no return value. 

Converting the other files follows the same pattern, with the exception of the `SearchForm` component, our single React class component. As a class component it needs to supply type variables to its base class, one for the type of its props, and one for the type of its state. 

The other interesting thing to note is that the type of `Search` props is specified as `SearchFormProps & SearchResults` which is the combination of those two types. In typescript this is called an [Intersection Type](https://www.typescriptlang.org/docs/handbook/advanced-types.html). 

```javascript
interface SearchFormProps {
    onSearch: (title: string) => void;
}

interface SearchResults {
    results: MovieSummary[];
}

class SearchForm extends React.Component<SearchFormProps, {title: string}> {
    constructor(props: SearchFormProps) {
        super(props);
        this.state = {title: ''};
        this.titleChange = this.titleChange.bind(this);
        this.search = this.search.bind(this);
    }

    titleChange(event: any) {
        this.setState({title: event.target.value});
    }

    search(event: any) {
        event.preventDefault();
        this.props.onSearch(this.state.title);
    }

    render() {
        return (
        <div><form onSubmit={this.search}>
            <label htmlFor="title">Title: </label>
            <input type="text" name="title" value={this.state.title} onChange={this.titleChange}/>
            <input type="submit" value="Search"/>
        </form>
        </div>
        );
    }
}

function Search({ onSearch, results = [] }: SearchFormProps & SearchResults) {
    return (
    <div>
        <h1>Search</h1>
        <SearchForm onSearch={onSearch} />
        <div>
            {results.map(({Title, Poster, imdbID}) =>
            <Link to={{type: 'MOVIE', payload: {imdbID}}} key={imdbID}>
                <img src={Poster} alt={Title} />
            </Link>)}
        </div>
    </div>
    );
}
```

Summary
=======

There is a typescript version of *create-react-app* that makes it easier to setup a new typed React application. Using a type checker to verify the type of props passed to React elements provides a guarantee within the component that the supplied data is correctly typed. 

Get The Code
------------

The code for this example is [on Github](https://github.com/liammclennan/movie-library-ts). You can access the code as it was at the completion of this step by cloning the repository and checking out the tag that corresponds to this post. 

```bash
git clone https://github.com/liammclennan/movie-library-ts.git
git checkout react10
```

or browse at https://github.com/liammclennan/movie-library-ts/tree/react10