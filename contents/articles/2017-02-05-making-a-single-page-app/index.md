---
title: Making a Single Page Application
author: Liam McLennan
date: 2017-02-05 20:32
template: article.jade
---

Should I?
=========

Conventional wisdom holds that in 2017 all applications are best implemented as single page applications (SPAs). Having seen the transformation from server-side MVC to client-side single page applications the cynical part of me wonders if the preference for SPAs has more to do with developer career planning that technical merits. 

Some hard truths. SPAs:

* are more complicated 
* take longer to build
* are much slower to render database content 

So, for me, applications that primarily publish content and have simple user interaction are much better as server side MVC applications. Applications requiring rich user interaction need to be SPAs. Everything else is somewhere in the middle. 

Get Started With A Starter Kit
--------------------

There are many prebuilt application starter kits available for building SPAs. They typically solve the modern nightmare of configuring a JavaScript build process and maybe throw in a few other goodies. 

For Angular there is:

* [the Angular Quickstart Seed](https://github.com/angular/quickstart). This an official Angular (Google) published quickstart repository. 
* [Angular cli](https://github.com/angular/angular-cli) includes a way to setup new projects. 

For React we have:

* [Mozilla Neo](https://github.com/mozilla/neo), a fully featured and opinionated application generator.
* [create-react-app](https://github.com/facebookincubator/create-react-app) the facebook idea of what a starter kit should be. Neat, but extremely minimal.   

There are hundreds of others available. Unfortunately, evaluating them is more difficult than you might think. The relative strenghts and weaknesses of these systems often don't present themself until a project is well underway, or when the developer decides that it is necessary to deviate in some way from the choices made by their chosen starter kit. 

I like React, and I think there is a lot of benefit in choosing the "official" starter kit creted and maintained by Facebook and the React team. The trouble with create-react-app is that it includes the absolute minimum required to make React work, which is not enough to build an application. For example, there is no routing, validation or application state management included. 

The best of both worlds then is to chose a minimal starter kit, upgrade it to include your favourite tools and match your preferences, then publish that as your own custom starter kit. This lets consumers use your starter kit as a single dependency in their applications, just as create-react-app does. It also means that starter kits are composable, and someone could further customise your starter kit into their own starter kit. 

What I Did To create-react-app
-----------------------

I used create-react-app as the basis for my starter kit. Then proceeded to add react-router for routing, bootstrap for styles and redux for state management. 

### Adding React router

The easy and obvious start is to install react-router:

```
npm install react-router --save
```

The fundamental abstraction of a web application is the resource (or page) identified by a URL. This is why routing is so important. Each application state should be addressable by an unique URL. If you don't have this then your application fails to meet the expectations of the web, such as a working back button, pages that can be bookmarked etc. To capture this I like to create page components that export everything required to hook a new page into the application, including a route string and a factory for creating the React component that represents the page. For example, the component for an about page might be:

```javascript
let AboutPage = {
    
    // function that returns the React component for the page
    pageFactory: aboutPageFactory,

    // the route to match to this page
    route: 'about'
};
export default AboutPage;
```

When rendering the application (this happens in `index.js` for create-react-app) we now have everything we need to connect routes to pages.

```javascript
ReactDOM.render(
    <Router history={browserHistory} >
        <Route path="/" component={Layout}>
            <IndexRoute component={Home}/>
            {[aboutPage, loginPage].map(page => 
                <Route key={page.route} path={page.route} component={page.pageFactory(store, page.sideEffects)} />
            )}    
        </Route>
    </Router>,
    document.getElementById('root')
```

For the about page, this means that the route `/about` will cause the about page component (returned from aboutPageFactory) to render within the `Layout` component. 