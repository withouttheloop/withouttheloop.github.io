---
title: The Timeline Backlog
author: Liam McLennan
date: 2016-09-02 20:32
template: article.jade
---

An agile product **backlog** is a prioritized list of uncompleted work. It can include features, bugs and other tasks. Good backlog items are as small as possible why still being *independently valuable*. They also have an associated effort estimate that helps the product owner to decide if and when the work should be completed. 

Prioritizing the backlog means sorting the items from highest priority to lowest priority. *Priority* is generally a combination of value to effort ratio and risk. That is, we should work first on the items that have the best "bang for buck" and the highest risk. High risk items come first so that the risk can be dealt with. Leaving all the risky items until the end of a project would make failure likely.

What is good about the Backlog?
=====

The product backlog is a central artifact for an agile project around which work is organised. It provides a basis for conversations with the product owner and subject matter experts. It guides the work of the team and enables project delivery to be projected into the future.

What is missing from the Backlog?
=====

The backlog doesn't communicate anything about time, and doesn't visualize relative effort. In short, the backlog is missing the kind of information that we might get from a timeline - so let's add one.

Consider the following fictitious backlog for a file backup application:

<style>
table>td,table>th { border: 1px solid }
</style>
<table border="1">
<thead>
<tr>
<th>Story</th>
<th>Effort (days)</th>
</tr>
</thead>
<tbody>
<tr>
<td>In order to not lose data as a computer user I want to be able to save backups of selected files</td><td>6</td>
</tr>
<tr>
<td>In order to not lose data as a computer user I want to have a configured portion of a filesystem automatically backed up continuously</td><td>12</td>
</tr>
<tr>
<td>In order to recover from a data failure as a computer user I want to be able to retrieve files that were backed up</td><td>10</td>
</tr>
<tr>
<td>In order to save money as a computer user I want old copies of backed up files to be purged</td><td>5</td>
</tr>
</tbody>
</table> 

<br/>Further suppose that we have a cost of $2000 / day and therefore a total cost estimate of $66,000. We'd like to see how that time and cost compares to our backlog. How much can I get for half that? Roughly when might a particular story be delivered? To answer these questions we can superimpose the backlog onto a timeline. 

Visualizing a backlog over time
======

Each story is shown as a row of a table, with the height scaled proportional to the effort for that story. If one story requires twice as much effort as another then its row is twice as high. A column to the right of the table shows how the budget is consumed over time, with label markers at 25%, 50% and 75% and 100%. This is enough detail to answer the timing questions listed above, but not so much detail that the product owner might infer that estimates have a higher precision than they really do.  

The resulting visualization is as follows:

<iframe width="100%" height="760" src="//jsfiddle.net/69z2wepo/54979/embedded/result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe> 

Interested parties can look at the timeline backlog and understand approximately when features will appear over time. Changes to the backlog are immediately understood as changes to the schedule. 