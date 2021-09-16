---
title: Spaced Repetition Wiki
author: Liam McLennan
date: 2015-01-10 20:32
template: article.jade
---

On the Wiki
----------

A wiki is a collection of user-editable web pages. The wiki was 'invented' by Ward Cunningham, so it must be good. I like to use wikis to document things, or record mind maps for areas that I am learning. I have a wiki for vim, a wiki for haskell, a wiki for linux and so on. 

Both github and bitbucket offer free wikis, and bitbucket wikis can be private. 

On Spaced Repetition
----------------

As we are repeatedly exposed to a datum the time before we forget it gets longer. The most efficient way then to commit something to memory is to be reminded of it immediately prior to the moment you would otherwise have forgotten it. You know this intuitively if you have ever studied for an exam and then realised three weeks later that you no longer retain any of the information. 

> Spaced repetition is a learning technique that incorporates increasing intervals of time between subsequent review of previously learned material in order to exploit the psychological spacing effect -- Wikipedia

Most of the things I learn I am happy to forget. It sounds strange but I know my memory is limited and [I believe that we tacitly retain helpful conditioning even when specific facts are forgotten](http://www.paulgraham.com/know.html). To be nerdy about it you could say that short-term memory trains our machine learning system. The memories fade but the prediction function remains. I like spaced repetition for those important data that I want to retain medium to long term. 

Combining the Wiki and Spaced Repetition
-----------------------

My struggle with spaced repetition has always been around the maintenance of the question decks. This problem has not existed for me with wikis, so I decided it might work well to embed spaced repetition questions in my wikis. The microformat I chose for this job is:

Q>>> This is the question? <<<

A>>> This is the answer. <<<

Any content at all (including HTML) is allowed between the delimiting tokens. Both this blog article and the [readme file for the project I created](https://raw.githubusercontent.com/liammclennan/SpacedRepetition/master/README.md) are valid examples of the format. The [aforementioned project](https://github.com/liammclennan/SpacedRepetition) is a terminal program, written in F#, that clones public git repositories, selects the set of markdown files, extracts the spaced repetition content that conforms to the specified microformat, and writes the data to a csv file that I can then import into [anki](https://raw.githubusercontent.com/liammclennan/SpacedRepetition/master/README.md) (my spaced repetition client of choice). 

Much of the program came together very nicely and provided some great demonstration of the joy of F#. The part I am least happy with, and the part I think most likely to contain bugs, is the [parser](https://github.com/liammclennan/SpacedRepetition/blob/master/SpacedRepetition/Parser.fs). I'm still learning parsers and parser combinators and this is the first one I've done that required backtracking. Regardless, it seems to work. 

<img src="glove.jpg" alt="hand in glove"/>

What's Next?
-----------

I plan to start some postgrad coursework soon and give this system a real workout. I may spin up a web front-end that does spaced repetition from public git repositories. The problems with existing spaced repetition clients are:

* proprietary formats
* closed systems
* non-web based

all of which I can easily fix with a simple web app. Currently the system is restricted to public git repositories as the data store but I could easily extend it to private git repositories or other data sources. I specifically chose git because it is an open standard. For this reason I abandoned the github and bitbucket APIs. 



