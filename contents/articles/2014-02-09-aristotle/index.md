---
title: Aristotle - A web based HTTP client
author: Liam McLennan
date: 2014-02-09 20:32
template: article.jade
---

Working with HTTP APIs I have found that there is a lack of GUI tooling to experiment and test requests. APIs are often documented as cURL commands, which is great, but cURL is a poor interface for experimenting. For example, if I want to post a document via cURL, I need to do something like:

    curl -vX POST https://api.github.com/users/liammclennan/repos 
        -H "content-type: application/json" -d '{"whatever": "value"}'

There are many options for testing HTTP APIs. Here some that I have tried.

Postman
-------

[Postman](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm?hl=en) is a chrome app. It is good for crafting HTTP requests. One nice feature is that it records all of the requests, so it is easy to reply past requests. I don't like that postman requires me to sign in to my google account to install it. Also, I have mostly stopped using Chrome for similar reasons.

Fiddler
------

[Fiddler](http://www.telerik.com/download/fiddler) is a windows GUI proxy that provides the ability to execute arbitrary HTTP requests. 

Aristotle
=========

[Aristotle](http://aristotle.onashirt.net/) exists because I want a HTTP client with the following features:

* easily execute a wide range of HTTP requests
* save and restore requests
* nicely format the response headers and body
* transform the response body and render views of the data
* cross-platform
* make requests to non-CORS-enabled APIs

After stuffing around for some time trying to design a nice interface for describing requests I realized that only developers need to experiment with HTTP APIs and therefore it is OK to require a bit of code to define the request, and jquery already has an API for defining a request that lots of developers are already familiar with. Therefore, when using Aristotle you define a request by calling $.ajax. The only actual requirement is to specify an expression that returns a jquery style promise that will be resolved with the right arguments. Here is one of the example requests templates used by Aristotle:

    $.ajax({
        type: 'GET',
        url: 'https://api.github.com/users/liammclennan/repos',
        dataType: 'json'
    });

Any request that can be specified as a call to $.ajax will work. Aristotle will then display the response headers and the response body as syntax highlighted json. 

But wait, there's more.

Often I am making requests to CouchDB and the results are difficult to interpret because there is too much data and json is not that easy to read. To solve this problem Aristotle allows you to define views. Corresponding to the above request to the github API Aristotle has a template to render the expected response. The code is:

    <%
    // manipulate the data some
    var mapped = data.map(function (repo) {
       return repo.name;
       });   
    %>

    <!-- define a template -->
    <ul>
        <% mapped.forEach(function(name) {%>
            <li><%=name%></li>
        <% }) %>
    </ul>

It firstly maps the response data to something more useful for the view and then uses an underscore template (similar to ejs) to render a list of my github repos. The output is

<ul>
  
      <li>angularserver</li>
        
            <li>author-quiz</li>
              
                  <li>backbone-basics</li>
                    
                        <li>browsertest</li>
                          
                              <li>Builder</li>
                                
                                    <li>coffeescript-course</li>
                                      
                                          <li>coffeescript-game-of-life</li>
                                            
                                                <li>coffeescript-koans</li>
                                                  
                                                      <li>couchpotato</li>
                                                        
                                                            <li>dbc</li>
                                                              
                                                                  <li>dddbrisbane13-fsharp</li>
                                                                    
                                                                        <li>dddbrisbane2012</li>
                                                                          
                                                                              <li>designbycontract</li>
                                                                                
                                                                                    <li>dotfiles</li>
                                                                                      
                                                                                          <li>eclipsewebsolutions.com.au</li>
                                                                                            
                                                                                                <li>functional-game-of-life</li>
                                                                                                  
                                                                                                      <li>gistblog</li>
                                                                                                        
                                                                                                            <li>git-deploy</li>
                                                                                                              
                                                                                                                  <li>giv.n</li>
                                                                                                                    
                                                                                                                        <li>googleajaxurls</li>
                                                                                                                          
                                                                                                                              <li>haskell-notes</li>
                                                                                                                                
                                                                                                                                    <li>Herald</li>
                                                                                                                                      
                                                                                                                                          <li>JavaScript-Koans</li>
                                                                                                                                            
                                                                                                                                                <li>JsBdd</li>
                                                                                                                                                  
                                                                                                                                                      <li>KeyRef</li>
                                                                                                                                                        
                                                                                                                                                            <li>Liam-s-Presentations</li>
                                                                                                                                                              
                                                                                                                                                                  <li>listagram_build</li>
                                                                                                                                                                    
                                                                                                                                                                        <li>mobile-dev</li>
                                                                                                                                                                          
                                                                                                                                                                              <li>node-test</li>
                                                                                                                                                                                
                                                                                                                                                                                    <li>onashirt</li>
                                                                                                                                                                                      
                                                                                                                                                                                      </ul>

Substitute any github user name in the url to get a list of that user's github repositories. 

What's Missing
-------------

Most things are missing. Proper error handling is missing. Saving requests is missing. Support for non-cors-enabled APIs is missing. Support for non-json responses is missing. A large selection of useful request templates is missing. 

I plan to keep working on Aristotle - mostly because it is a tool that I want to use. 
