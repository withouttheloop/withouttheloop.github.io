---
title: Authentication for the modern web
author: Liam McLennan
date: 2017-07-21 20:32
template: article.jade
---


![Xerox PARC](parc.jpg)

April 1971, here at Xerox PARC, the Austrian-born computer scientist Hanz Scmitfilgerz connects to the latest in mainframe computer technology. 

To establish his identity to the satisfaction of the computer he authenticates by providing a combination of: a username that identifies him, and a password that only he knows. His password is a unique, long, high-entropy string of characters known only to Dr Scmitfilgerz, and stored only in his memory. 

![mainframe](mainframe.jpg)

That pleasent spring morning of 1971 is the last time that password authentication worked well for anyone. 

Why passwords are not secure
====================

Real users are not like Dr Scmitfilgerz. They can't be in an age where the *average* person has dozens if not hundreds of online user accounts. It is beyond our capability to remember all those passwords so, people typically 

* reuse the same password or 
* store their passwords somewhere

Reusing passwords is perhaps the most significant password security problem today. At regular intervals a large password database is compromised and released on the internet (recent examples include linkedin, adobe, sony). Criminals then take those public credentials and attempt, often successfully, to access other services such as email or online banking. 

Storing passwords can be OK, but it does introduce a massive vulnerability. If the mechanism used to store your passwords is compromised then all of your accounts are compromised. An incident like this is unlikely but catastrophic. 

Why passwords are a bad user experience
==============================

IT professionals often solve the password proliferation and storage problem with an online product like Lastpass or 1Password. For the typicall consumer internet user these products are confusing and too hard to use. I will never be able to train my parents to use a password manager. 

Assuming we do not reuse passwords (for the reason detailed above) then each time I want to access a new service I have the additional burden of coming up with a new password, and remebering it somehow. I have to fight my way through the services password complexity tests (your pasword must include lower case, uppercase, numeric characters and a gang sign). 

It is too hard. Non-technical people don't do it. They reuse the same awful password (like 'password') or variations of the same password.

Modern authentication (Authentication 2.0)
====================================

Recognizing the limitation of password-based authentication, some organisations like [medium](https://blog.medium.com/signing-in-to-medium-by-email-aacc21134fcd) and [Slack](https://slack.com/) are abandoning passwords in favour of email-based authentication. Users are authenticated by proving they have access to an email inbox. In practice, the workflow is:

1. A user visits a site they wish to login to
1. They enter their email address
1. The site sends a special "magic link" to their email address
1. The user accesses their email, clicks on the "magic link" and is authenticated

This workflow is simple and fast. For most users it is vastly more secure than a password based system and easier to use. For website owners it [results in more signups](https://komo.tech/increase-registrations) and less password related support. 

Introducing [Komo](https://komo.tech)
==============

A number of providers offer a hosted email authentication service that website owners can use to add passwordless authentication to their site. I've built another ([Komo](https://komo.tech)) that offers a slightly different service.

1. It is focused purely on passwordless authentication, without all the expensive add-ons that usually comes with Identity and Access Management (IAM) SaaS providers. 
1. The core functionality of passwordless email authentication is, and always will be, completely free (with reasonable volume limits).
1. Komo is designed to integrate properly with your web framework's authentication support. It is not an awkward JavaScript bolt-on. 

<a href="https://komo.tech">![Komo passwordless authentication](komo.png)</a>

Komo is available and ready to provide email authentication for your site. Just [sign up](https://komo.tech/signup?returnUrl=/dashboard). The Komo login process uses Komo for authentication (of course). 

You can also try a [a demo site](https://immense-sneeze.glitch.me/) that uses Komo for login, and [browse the source](https://glitch.com/edit/#!/immense-sneeze) to see how it is implemented.

