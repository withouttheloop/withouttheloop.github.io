---
title: snap
author: Liam McLennan
date: 2012-02-04 20:32
template: article.jade
---

<a href="http://withouttheloop.com" border="0"><img src="https://github.com/liammclennan/gistblog/raw/master/public/img/return-button.png" align="right" hspace="10" vspace="10" alt="return to withouttheloop.com"></a>

> This file is a fantasy readme for an tool that doesn't exist yet. I wrote it in an attempt to apply [readme driven development](http://tom.preston-werner.com/2010/08/23/readme-driven-development.html). The idea is to force yourself to define a vision for your product/tool/whatever before starting to build anything.


Snap
====

snap is a tool that automatically reloads your browser based on filesystem events. For example, it can be configured to automatically reload the webpage you are working on when any of your html files are modified.

```js
snap({
  '.': /.+\.html/
});
```

Usage
-----

Create a snap script (as above) and start the server:

    node snap mysnapscript.js

Add the following script to the page that you want auto-reloaded:

```
<script src="http://localhost:9876/snap.js"></script>
```