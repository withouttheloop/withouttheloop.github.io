---
title: Sharing client and server validation
author: Liam McLennan
date: 2012-02-14 20:32
template: article.jade
---

Client-side web applications (SPAs whatever) must perform client-side validation to achieve the required responsiveness. But the client is not a secure environment, so client-side validation is a convenience, not to be trusted. Validation has to be performed server-side to ensure security and data correctness. 

Traditional Solutions to the Client / Server Validation Problem
---------------

### Shared Metadata

Asp.net mvc exports validation metadata from server-side models and embeds it into form markup. This process is distributed throughout mvc's form helpers. Each time a form element is generated the framework adds data- attributes to that element that describe that element's validation. 

On the client-side a script runs that reads the validation metadata from the data- validation attributes and translates that metadata into a format suited to a client-side validation framework (asp.net mvc uses jquery.validation).

By sharing metadata between the server and the client it is possible to have a validation framework on each side automatically working with the same metadata.

### Doing It Twice

The solution that I am currently using is to implement completely separate validation - once on the client and once on the server. It works, but it is frustrating and boring. There is also a maintenance burden in keeping the validations synchronized.

Moving Client-Side Validation to the Server
---------------------

The idea I have been considering is to move the client-side validation to the server. Client-side validation tests javascript objects against validation metadata and builds a set of results. Why can't we do that on the server?

My idea is to write a custom model binder that binds json data posted to the server. The post would have to include metadata that defines the schema of the data, so that we can lookup the required validation metadata. A JavaScript interpreter can then be used to apply the validation. 

The strength of this idea is that it uses the same validation engine and validation metadata on the client and the server. The validation only needs to be tested once and cannot possibly get out of sync.
