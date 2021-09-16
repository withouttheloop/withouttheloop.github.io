---
title: HTTP Basic Authentication Support in Wurl
author: Liam McLennan
date: 2014-10-08 20:32
template: article.jade
---

[Wurl](http://wurl.io) (web curl) is a browser-based HTTP test client that I [slapped together earlier this year (2014)](http://withouttheloop.com/articles/2014-03-11-wurl/), partly because it is a useful tool and partly because it gives me a play project to experiment with React, Typescript and F#. 

How it works
-------------

Wurl describes http requests using the jquery [$.ajax syntax](http://api.jquery.com/jquery.ajax/). I chose this because most developers are familiar with it and because it is capable of describing a wide variety of request options.

Wurl first attempts to make the request by directly evaluating the request using jquery from the browser. Try [this fun example](http://wurl.io/request?url=https%3A%2F%2Fapi.github.com) of querying the github API. Often this strategy will fail due to [cross-origin resource sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS) restrictions. Wurl is smart enough to detect CORS errors and fallback to making the request via a proxy, which is not subject to CORS restrictions. 

Unfortunately the proxy is not jquery and does not use jquery (it uses [HttpClient](http://msdn.microsoft.com/en-us/library/system.net.http.httpclient%28v=vs.118%29.aspx)) so the query must be parsed and translated. Therefore queries that go via the CORS proxy do not implement the full set of $.ajax features. 

HTTP basic authentication
---------------

Recently I added support for the Authorization header. The options that are honoured now are: url, type (http verb), headers (Authorization only). This means that basic authentication can be used from Wurl for all services, regardless of CORS support.  

<iframe src="http://wurl.io/template/d3ba57f1-ebd4-44a6-ad8c-729881cd495f" style="width: 100%;height:400px;" frameborder="0"></iframe>

Here is an [example showing a request to the github API that uses basic authentication](http://wurl.io/template/d3ba57f1-ebd4-44a6-ad8c-729881cd495f). It fails, but only because I have removed my password. 
