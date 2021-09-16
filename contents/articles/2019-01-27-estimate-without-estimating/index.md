---
title: How To "Estimate" Without Estimating
author: Liam McLennan
date: 2019-01-27 20:32
template: article.jade
---

The trouble with estimation is that it is both essential and impossible. 

For some time one of the great twitter battlegrounds has been #NoEstimates. The clickbaity hashtag encourages the two sides to talk past each other and claim intellectual superiority. For instance:

> "The #NoEstimates advocates are essentially uninformed about basic probabilistic decision making and willfully ignores the principles to be found easily on the web A middle schooler with Google knows more than they do"

Nice one. Here comes the reply!

> "That argument is bullshit. But I'll let it slide because you obviously have read anything at all about #NoEstimates Start here. 6 authors, 20+ blog posts http://softwaredevelopmenttoday.com/noestimates"

The trouble with estimation is that it is both essential and impossible. It is also harmful. There is a battle tested way to get the benefits of estimation, without the problems of actually doing it. But first, some context. 

How Estimation is Essential
-------------------

Estimation is essential. My family are considering renovating our house. If the builder told me he would like to start the project, measure velocity and then give me a projection of possible costs our conversation would be over. I don't have unlimited funds and I cannot sponsor an unlimited construction budget. Similarly, my consulting clients inevitably want to know roughly how much they need to invest to get their problem solved. It is not fair to simply say that estimation is hard and the work will be finished when it is finished. 

How Estimation is Impossible
-------------------

I am tempted to simply state this one as an axiom, however, I know there are people who disagree. I have been a professional software developer for 20 years and have never met anyone who could estimate accurately. The way it is frequently simulated is simply to estimate far too high, then keep working until you hit your estimate. Magic! If a team goes over an estimate it is visible, if they are under the estimate they appear exactly on time. 

That's just an anecdote. Good point. Here are some more:

> "Estimates are difficult. When requirements are vague — and it seems that they always are — then the best conceivable estimates would also be very vague. Accurate estimation becomes essentially impossible. Even with clear requirements — and it seems that they never are — it is still almost impossible to know how long something will take, because we’ve never done it before. If we had done it before, we’d just give it to you." -- [Ron Jeffries](http://ronjeffries.com/xprog/articles/the-noestimates-movement/)

> "Every project contains uncertainty. If it doesn’t you shouldn’t be doing it — it’s already been solved!" -- [Dan North](https://dannorth.net/2013/08/08/blink-estimation/)

If you prefer data, the CHAOS report has found consistently that 75% of projects are late/over budget or failed. Even a proponent of estimates, Steve McConnell, says, "we have come to value project control over project estimation, as a means of achieving predictability".  

All of that is to say why we cannot predict the amount of effort required to deliver a particular, defined feature. Here is an even bigger problem. People want estimates before they commit to an initiative. In the agile world an initiative is a problem to solve - there is no particular defined feature. Unless we are doing big-design-up-front and prematurely committing to a solution of a problem not yet well understood there isn't even anything to estimate. It's like asking my builder to accurately estimate my renovation defined as "more space and brighter colours". 

> "Predefined backlogs of work reduce creativity and inhibit steering the project for success." -- [Ron Jeffries](https://pragprog.com/magazines/2013-04/estimation)

How Estimation is Harmful
-------------------------

Return on investment depends upon cost and value. The focus on estimation puts the emphasis entirely on the cost side. When was the last time someone asked for an estimate that included an estimate of value? Teams that focus on estimates tend to track progress against cost, but do they track progress against value? (see [Project Lamb Meatballs - Value-based vs Project-based Portfolio Management](https://withouttheloop.com/articles/2018-12-12-project-lamb-meatballs/))

As high performing teams move from managing outputs to managing outcomes it is natural to shift emphasis from cost towards value. Ron Jeffries agrees, "teams focused on estimation are generally working the short end of the lever of cost vs. value. Better teams focus more on value and less on cost". 

Estimates encourage a zero-sum mindset. On the customer side I want to get as much as I can for as little as possible to demonstrate my superior negotiation capability. Teams get beaten up over their estimates, which were impossible to start with. Then they cut corners and find ways to get back to the estimate, at the cost of creating real value. Estimates are disputed and argued over, then when a requirement changes, the battle starts all over again. 

Perhaps most importantly, estimation locks us into a path (the one we estimated), chosen at a time when we had the least possible information (before we started). Intelligent, motivated professionals are unable to pursue to best outcome because they are locked into a specific course of action. In theory it is possible to significantly renegotiate the agreed scope. In practice people tend to just stick to the plan. 

How Estimation is Helpful
-------------------------

I once built a deck. I am not a builder and I did not know how to build a deck, therefore I accepted that estimation was impossible. I knew enough to know that the raw materials would be less than $10,000. The other investment was my labour, which I was prepared to contribute until the job was done. I did not estimate the project, but I assembled enough information to support the investment decision. I wanted a deck and I got one within an acceptable cost. 

How to Estimate Without Estimating
--------------------------

Here is how this might play out in a consulting scenario:

Me: "Hi, I'd like a deck please! How much will that cost?"

Builder: "'Deck' does not precisely define what you want, so I cannot provide an exact figure. Looking at your house, and knowing the type of decks people tend to build for this type of property and in this location I can build you a deck for $10,000. If you'd like a really fancy deck then budget $15,000. I am confident that I can deliver "a deck" within these parameters."

Me: "Woo hoo!"

The key here is in the vagueness of the project definition: "a deck". This allows the builder to manage the project to the budget. If decking is expensive the week the project commences she can build a more modest deck that still meets my needs. 

As the sponsor of an initiative what I really need is to know that I will get **A SOLUTION** for **A PRICE** (or price range). By leaving the precise solution undefined I:

* make it possible to reliably find a solution within the budget
* leave the experts delivering my solution the freedom to do their jobs to the best of their abilities

Many industries operate this way. You pay your mechanic $X to fix your car. You pay your lawyer $Y to protect your intellectual property. You don't specify **how** they should do it, because they are the experts.

While I might pay a trusted builder to construct "a deck" nobody will pay for "an application". To make this approach specific to software development we need a better way to clarify that the customer will get a solution to their problem (**a** solution, not **the** solution). 

![a solution not the solution](fight_club_airport_security.jpg)

I like a technique called goal-based requirements analysis. In short, all actors in the system are identified along with their goals. An acceptable solution is one which allows all actors to achieve their goals. That is all an initiative sponsor really wants or needs, and at this level of detail we **can** nominate an appropriate budget. 

Summary
-------

1. Clarify the problem to be solved.
1. Work to establish a sufficient budget.
1. Agree to provide the best solution you can, within the budget agreed.
1. Follow the **outcome** not the **output**. 