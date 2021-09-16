---
title: Learning Cycle.js
author: Liam McLennan
date: 2015-09-03 20:32
template: article.jade
---

<img src="cyclejs_logo.svg" alt="Cycle.js logo" style="width:250px;" align="right">

[Cycle.js](http://cycle.js.org/) is clearly React inspired, but it attempts to fix some areas where React is deficient, or weird. It also advocates (forces?) a functional style, composing immutable state via observables.

Even with a React background the learning curve is steep. I started to see how it is possible to reason about Cycle.js applications when I saw [the refactoring to the Intent-Model-View pattern](https://meet.lync.com/readify-net/liam.mclennan/LW6W5FMZ). See [jsbin](http://jsbin.com/vikaga/1/edit?js,output).

This post includes graphics and code samples from the [Cycle.js](http://cycle.js.org/) website.

Intent
=====

Intent is typically a function from sources of input (drivers) (DOM, HTTP, etc) to a set of observables, named according to the user's intent (e.g. `changeHeight$`, `receiveUserDetails$`).

```
function intent(DOM) {
  return {
    changeWeight$: DOM.get('#weight', 'input')
      .map(ev => ev.target.value),
    changeHeight$: DOM.get('#height', 'input')
      .map(ev => ev.target.value)
  };
}
```

Model
====

Model is a function from intents (produced by the `Intent` function) to a `state$` observable. The latest value of the `state$` observable is the current state of the component. `state$` is observed by the view, so changes to `state$` cause changes to the view. Earlier examples had the state spread across multiple observables (e.g. one for the DOM, one for each possible HTTP request). Consolidating state into a single observable takes some work but makes the component easier to reason about and the view easier to implement.

```
function model(actions) {
  return Cycle.Rx.Observable.combineLatest(
    actions.changeWeight$.startWith(70),
    actions.changeHeight$.startWith(170),
    (weight, height) =>
      ({weight, height, bmi: calculateBMI(weight, height)})
  );
}
```

View
====

View is a function from the model observable (`state$`) to a vdom observable. This is equivalent to React's `Render` function.

```
function view(state$) {
  return state$.map(({weight, height, bmi}) =>
    h('div', [
      renderWeightSlider(weight),
      renderHeightSlider(height),
      h('h2', 'BMI is ' + bmi)
    ])
  );
}
```

main()
====

<svg  style="width:100px;" viewBox="0 0 209 507" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:sketch="http://www.bohemiancoding.com/sketch/ns">
    <!-- Generator: Sketch 3.3.2 (12043) - http://www.bohemiancoding.com/sketch -->
    <title>Group</title>
    <desc>Created with Sketch.</desc>
    <defs></defs>
    <g id="human-computer" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd" sketch:type="MSPage">
        <g id="Group" sketch:type="MSLayerGroup" transform="translate(-45.000000, -15.000000)">
            <circle id="Oval-1" stroke="#9AC572" stroke-width="10" fill="#A5FF51" sketch:type="MSShapeGroup" cx="146" cy="419" r="38"></circle>
            <circle id="Oval-2" stroke="#B9B9B9" stroke-width="10" fill="#DDDDDD" sketch:type="MSShapeGroup" cx="146" cy="140" r="38"></circle>
            <text id="Human" sketch:type="MSTextLayer" font-family="Source Sans Pro" font-size="50" font-weight="normal" sketch:alignment="middle" fill="#7DA25A">
                <tspan x="68.4" y="521">Human</tspan>
            </text>
            <text id="Computer" sketch:type="MSTextLayer" font-family="Source Sans Pro" font-size="50" font-weight="normal" sketch:alignment="middle" fill="#747474">
                <tspan x="43.7" y="49">Computer</tspan>
            </text>
            <g id="Path-15-+-Path-2" transform="translate(98.500000, 282.000000) scale(1, -1) translate(-98.500000, -282.000000) translate(75.000000, 183.000000)" stroke="#979797" sketch:type="MSShapeGroup">
                <path d="M37.3775229,1.50509088 L16.9737944,18.4291738 L37.3775229,34.2764494" id="Path-15" stroke-width="7" transform="translate(27.175659, 17.890770) rotate(125.000000) translate(-27.175659, -17.890770) "></path>
                <path d="M26.9129005,197.154998 C26.9129005,197.154998 -34.8157014,98.8736491 31,12.3678484" id="Path-2" stroke-width="9"></path>
            </g>
            <g id="Path-15-+-Path-3" transform="translate(194.500000, 279.000000) scale(-1, 1) translate(-194.500000, -279.000000) translate(171.000000, 180.000000)" stroke="#9BC672" sketch:type="MSShapeGroup">
                <path d="M37.3775229,1.50509088 L16.9737944,18.4291738 L37.3775229,34.2764494" id="Path-15" stroke-width="7" transform="translate(27.175659, 17.890770) rotate(125.000000) translate(-27.175659, -17.890770) "></path>
                <path d="M26.9129005,197.154998 C26.9129005,197.154998 -34.8157014,98.8736491 31,12.3678484" id="Path-2" stroke-width="9"></path>
            </g>
        </g>
    </g>
</svg>

A cycle.js application includes a function that represents the computer side of the human-computer interaction, often called `main`. It is a function from input drivers (http responses, DOM events) to output drivers (http requests, vdom structure). The [Get Random User](http://jsbin.com/yizisa/6/edit?js,output) example shows working with two drivers (HTTP and DOM).

```
function main({DOM}) {
  return {DOM: view(model(intent(DOM)))};
}
```

Cycle.run()
======

An application is started by calling `Cycle.run()` specifying the `main` function and the required drivers. E.g.

```
Cycle.run(main, {
  DOM: makeDOMDriver('#app'),
  HTTP: makeHTTPDriver()
});
```
