---
title: My open source projects
author: Liam McLennan
date: 2012-02-11 20:32
template: article.jade
---

I have a lot of projects on github now, some active and some archival. This post is a short summary of the projects that I actively maintain. 

watchn
------

[https://github.com/liammclennan/watchn](https://github.com/liammclennan/watchn)

Watchn is a node-based file system watcher. It can be used to trigger actions when specific file system changes are detected. I use it to drive [continuous testing](http://blog.objectmentor.com/articles/2007/09/20/continuous-testing-explained) and for automating build tasks.

git-deploy
----------

[https://github.com/liammclennan/git-deploy](https://github.com/liammclennan/git-deploy)

git-deploy is a simple git based deployment tool. Push changes to a deployment branch and click a button to deploy. git-deploy can also run pre and post deployment scripts.

snap
----

[https://github.com/liammclennan/snap](https://github.com/liammclennan/snap)

snap is an automatic browser reloader for live testing web pages. It uses watchn to detect file system changes, which then cause the browser to reload. Useful for web development and for browser based testing. Similar to livereload except its free and works on windows.

gistblog
--------

[https://github.com/liammclennan/gistblog](https://github.com/liammclennan/gistblog)

gistblog is my gist-based blog engine that powers [my blog](http://withouttheloop.com/). 

backbone-server
---------------

[https://github.com/liammclennan/backbone-server](https://github.com/liammclennan/backbone-server)

I use it for backbone but backbone-server is actually a generic non-persistent RESTful http server. Start the server and begin storing and retrieving resources. Data is stored in memory so it is lost when the server stops. Useful for testing and teaching backbone. See [the demo](http://jsfiddle.net/ynkJE/24/).

JavaScript Koans
----------------

[https://github.com/liammclennan/JavaScript-Koans](https://github.com/liammclennan/JavaScript-Koans)

JavaScript koans is an interactive learning environment that uses failing tests to interactively teach JavaScript.

