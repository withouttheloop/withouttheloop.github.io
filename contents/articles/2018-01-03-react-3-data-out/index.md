---
title: Learning React 3 - Data out with props
author: Liam McLennan
date: 2018-01-03 20:32
template: article.jade
---

[Learing React 2 - Data in with props](/articles/2018-01-03-react-2-props/) showed how to reuse a component by passing data in to that component. This post will show how to pass data back out of a component, again using props.

> Data is passed out of a React component by calling functions passed in as props

Recall our `Today` component:

```javascript
function Today({ day }) {
    return <div>Today is {day.toString()}</div>;
}
```

We add a button:

```javascript
function Today({ day }) {
    return <div>
                Today is {day.toString()}
                <button type="button">Next</button>
            </div>;
}
```

Now our UI has a button. Let's modify the usage of `Today` to pass a function that will be called when the 'Next' button is clicked:

```javascript
function onNext() {
    console.log('Next!');
}

ReactDOM.render(<Today day={new Date(2018, 2, 20)} onNext={onNext} />, 
    document.getElementById('root'));
```

and connect this to the `onClick` event of the button:

```javascript
function Today({ day, onNext }) {
    return <div>
                Today is {day.toString()}
                <button type="button" onClick={onNext}>Next</button>
            </div>;
}
```

Now when the button is clicked the text 'Next!' is written to the console. By passing arguments to the `onNext` function we have a mechanism for getting events and data out of our component.

Now we can use the `onNext` event to modify the `day` value and update the UI, completing the circle of data flow.

```javascript
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

Note that I had to move the call to `ReactDOM.render` into a function and call that function after updating the value of `day`. This is because React does not automatically now that its model data has changed and it needs to re-render. 

Summary
=======

Data is passed into a React component as a value supplied as props. Data is passed out of a React component by calling a function passed in as props. Components created in this style have a wonderful simplicity. Their interface is defined entirely by props. You should try to code all of your React components in this way. 

> Next: [Learning React 4 - Redux](/articles/2018-01-05-react-4-redux/)

Get The Code
------------

The code for this example is [on Github](https://github.com/liammclennan/today-react-app). You can access the code as it was at the completion of this step by cloning the repository and checking out the tag that corresponds to this post. 

```bash
git clone https://github.com/liammclennan/today-react-app.git
git checkout react3
```

or browse at https://github.com/liammclennan/today-react-app/tree/b59c3dbeb3597395d6c0c7d9b2831bf3999e870a.
