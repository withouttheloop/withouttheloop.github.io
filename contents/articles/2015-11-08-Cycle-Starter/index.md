---
title: Cycle.js Starter
author: Liam McLennan
date: 2015-11-08 20:32
template: article.jade
---

Just getting a [cycle.js](http://cycle.js.org/) app building can take some time. [This repo](https://github.com/liammclennan/cycle-starter) is meant to provide a working starting point for a cycle.js app using Babel to compile ES2015 to ES5, webpack to bundle the modules into a script and gulp to orchestrate the build process.

The build produces a single bundled script in browser/bundle.js.

Get Started
===========

    git clone https://github.com/liammclennan/cycle-starter.git

Install gulp:

    npm install -g gulp

Install dependencies:

    npm install

Build:

    gulp

Then open `index.html` in your browser.
