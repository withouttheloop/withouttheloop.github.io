---
title: Nginx Lets Encrypt SSL Reverse Proxy for Azure Web Apps
author: Liam McLennan
date: 2017-07-23 20:32
template: article.jade
---

In 2017, deploying a new web site realistically means having a custom domain name and secure connection (SSL). Unfortunately for Microsoft Azure users custom domains and SSL are two of the segmentation triggers that push you out of the free tier and into higher priced plans. 

Without a custom domain name or SSL I can host my application for free. Adding a custom domain name will cost $15/month extra for hosting (plus the cost of the domain and DNS). SSL support pushes me to $54/month for very similar hardware (1 CPU, 1.75GB RAM). 

My Situation
============

I wish to deploy a .net application written in F#. At the time of writing deploying F# to linux is not a good option as I am not prepared to waste huge amounts of time. Therefore, the simplest hosting option is to use an Azure Web App. I can deploy by pushing to a git branch and the hosting will just work for a .net application. 

To avoid the costs described above in configuring Azure for a custom domain and SSL I thought I'd try using an existing nginx server as a reverse proxy.

My Solution
===========

DNS
----

Firstly, I purchased a domain name (`komo.tech`) and configured the DNS to point the domain name to a Digital Ocean Linux server running nginx. 

nginx
----

Next, I added a new site to the nginx configuration. The site won't actually host any local content it simply receives requests and forwards them on to the Azure Web App, thus, it is a reverse proxy. 

In the nginx configuration I specified the underlying host that nginx will be proxying for. 

```
upstream app_komo {                           
    server simpleauth1.azurewebsites.net:443; 
}                                             
```

`simpleauth1.azurewebsites.net` is the domain name of the Azure Web App. `443` is the port to use because I want a secure connection. Later I will specify the protocol.

Next, I created a `location`, telling nginx to proxy all requests, from the root (`/`) to the previously declared upstream (the Azure Web App):

```
location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host simpleauth1.azurewebsites.net;
    proxy_set_header X-Forwarded-Proto $scheme;
    #proxy_set_header X-NginX-Proxy true;

    proxy_ssl_session_reuse off;
    proxy_pass https://app_komo/;
    proxy_redirect off;
}
```

At first my configuration did not work. I got an error from Azure about the site not existing. This was happening because the forwarded requests were arriving at Azure as requests for `komo.tech`, a domain that Azure knows nothing about. The solution was to add the line `proxy_set_header Host simpleauth1.azurewebsites.net;`. This means that when a request passes through the reverse proxy the host is changed from `komo.tech` to `simpleauth1.azurewebsites.net`, a domain that Azure knows how to handle. 

https (SSL) with Let's Encrypt
----------

The final step was to secure the connection between user's browsers and the reverse proxy with SSL. To do this I used the free service [Let's Encrypt](https://letsencrypt.org/). For my setup (nginx on Ubuntu 14.04) the steps were:

1. Install certbot with apt-get
1. Run certbot `certbox --nginx` and follow the prompts to specify which sites require certificates.
1. Add nginx configuration to redirect http traffic to https 
    ```
    server {                                       
        listen 80;                                 
        server_name komo.tech;                     
        return 301 https://$host$request_uri;      
    }                                              
    ```
1. Restart nginx 

Conclusion
=========

It is not the ultimate configuration by an means, but proxying via nginx is simple and effective. It also saved me some hosting cost. 