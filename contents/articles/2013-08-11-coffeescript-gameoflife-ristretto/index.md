---
title: CoffeeScript Game of Life with Ristretto Type Annotations
author: Liam McLennan
date: 2013-08-11 20:32
template: article.jade
---

This weekend at [campjs](http://campjs.com/) one of my projects has been porting my [Haskell Game of Life](http://withouttheloop.com/articles/2013-05-12-functional-game-of-life-3-haskell/) to CoffeeScript, with type annotations courtesy of [Ristretto](http://code.google.com/p/ristretto-js/). 

[https://github.com/liammclennan/coffeescript-game-of-life](https://github.com/liammclennan/coffeescript-game-of-life)

Ristretto
-----

Given a function from string to int:

    number_of_chars = (s) -> s.length

Ristretto can enfore the types given the following annotated version:

    number_of_chars = T 'number_of_chars :: String -> Int', (s) ->
        s.length

The format of the type annotation will be familiar to Haskellers. Read `::` as 'has the type'. All types to the left of a -> are function arguments and the type to the right of the last -> is the return type. So the above function `number_of_chars` has the type `String` to `Int`. 

Ristretto can also define data types. For example:

    T 'typedef Person :: { name: String, age: Int }'
    class Person
        constructor: (@name, @age) ->

The JavaScript equivalent is:

    T('typedef Person :: { name: String, age: Int }');
    function Person(name,age) {
        this.name = name;
        this.age = age;
    }

Game of Life
------

Here is my Game of Life implementation:
    

    require 'coffee-script'
    T = require './ristrettoAssert'
    _ = require 'underscore'

    # Define a type for a living cell
    T 'typedef Cell :: {x:Int, y:Int}'
    class Cell
        constructor: (@x,@y) ->

    # seed the initial living cells
    r_pentomino = [new Cell(2, 1), new Cell(3, 1), new Cell(1,2), new Cell(2,2), new Cell(2,3)]

    # determine if a coordinate (x,y) is a neighbour of a cell
    is_neighbour = T('is_neighbour :: Cell -> Int -> Int -> Bool', (c, x, y) ->
        Math.abs(x-c.x) < 2 and Math.abs(y-c.y) < 2 and (not (x is c.x and y is c.y))
    )

    living_neighbours = T('living_neighbours :: [Cell] -> Int -> Int -> Int', (cs,x,y) ->
        (cell for cell in cs when is_neighbour cell, x, y).length
    )

    is_on = T('is_on :: [Cell] -> Int -> Int -> Bool', (cs,x,y) ->
        (cell for cell in cs when cell.x is x and cell.y is y).length > 0
    )

    largest_x = T 'largest_x :: [Cell] -> Int', (cs) ->
        _.max(cs, (i) -> i.x).x

    largest_y = T 'largest_y :: [Cell] -> Int', (cs) ->
        _.max(cs, (i) -> i.y).y

    alive_next_generation = T 'alive_next_generation :: [Cell] -> Int -> Int -> Bool', (cs,x,y) ->
        false
        (is_on(cs, x, y) and 2 <= living_neighbours(cs, x, y) <=3) or ((not is_on(cs, x, y)) and (living_neighbours(cs,x,y) is 3))

    tick = T 'tick :: [Cell] -> [Cell]', (cs) ->
        ncs = []
        for x in [0..largest_x(cs)+1]
            for y in [0..largest_y(cs)+1]
                ncs.push(new Cell(x,y)) if alive_next_generation cs, x, y
        ncs

    visualize = T 'visualize :: [Cell] -> String', (cs) ->
        result = ''
        for y in [0..largest_y(cs)+1]
            for x in [0..largest_x(cs)+1]
                result += if is_on(cs,x,y) then 'x' else '-'
                if x is largest_x(cs)+1
                    result += '\n'
        result

    [1..50].reduce((p,c,i) ->
        console.log visualize p
        tick p
    , r_pentomino)

Thoughts
-----

While re-implementing game of life my experience was that ristretto frequently alerted me to type errors. Without Ristretto these errors would have manifested later in the program with a greater distance between the problem and the runtime error. 

Ristretto does not provide the full benefits of a static type checker. It is more like a type-based [design-by-contract](http://en.wikipedia.org/wiki/Design_by_contract) library. Personally, I have been frustrated by type errors in Javascript, which led me to implement a [basic runtime type checker for JavaScript](https://github.com/liammclennan/dbc), but I think that Ristretto is a better and more logically consistent solution.

Output
-----

The output of my CoffeeScript Game of Life for the first 50 generations takes 22s on my Chromebook and is printed below. To generate the full 1103 generations before this pattern stabilises will likely require a better algorithm. 

    -----
    --xx-
    -xx--
    --x--
    -----

    -----
    -xxx-
    -x---
    -xx--
    -----

    --x--
    -xx--
    x--x-
    -xx--
    -----

    -xx--
    -xxx-
    x--x-
    -xx--
    -----

    -x-x-
    x--x-
    x--x-
    -xx--
    -----

    --x---
    xx-xx-
    x--x--
    -xx---
    ------

    -xxx--
    xx-xx-
    x--xx-
    -xx---
    ------

    xx-xx-
    x-----
    x---x-
    -xxx--
    ------

    xx----
    x--xx-
    x-xx--
    -xxx--
    --x---
    ------

    xx----
    x--xx-
    x-----
    ------
    -xxx--
    ------

    xx--
    x---
    ----
    -xx-
    --x-
    --x-
    ----

    xx---
    xx---
    -x---
    -xx--
    --xx-
    -----

    xx---
    --x--
    -----
    -x-x-
    -xxx-
    -----

    -x---
    -x---
    --x--
    -x-x-
    -x-x-
    --x--
    -----

    -----
    -xx--
    -xx--
    -x-x-
    -x-x-
    --x--
    -----

    -----
    -xx--
    x--x-
    xx-x-
    -x-x-
    --x--
    -----

    ------
    -xx---
    x--x--
    xx-xx-
    xx-x--
    --x---
    ------

    ------
    -xx---
    x--xx-
    ---xx-
    x--xx-
    -xx---
    ------

    -------
    -xxx---
    -x--x--
    --x--x-
    -x--x--
    -xxx---
    -------

    --x----
    -xxx---
    -x--x--
    -xxxxx-
    -x--x--
    -xxx---
    --x----
    -------

    -xxx---
    -x-x---
    x----x-
    xx---x-
    x----x-
    -x-x---
    -xxx---
    -------

    -x-x----
    xx-xx---
    x-x-x---
    xx--xxx-
    x-x-x---
    xx-xx---
    -x-x----
    --x-----
    --------

    xx-xx-
    x---x-
    --x---
    x-x-x-
    --x---
    x---x-
    xx-xx-
    --x---
    ------

    xx-xx-
    x-x-x-
    ------
    --x---
    ------
    x-x-x-
    xxxxx-
    -xxx--
    ------

    xxxxx-
    x-x-x-
    -x-x--
    ------
    -x-x--
    x-x-x-
    x---x-
    x---x-
    --x---
    ------

    x-x-x--
    x---x--
    -xxx---
    -------
    -xxx---
    x-x-x--
    x---xx-
    -x-x---
    -------

    -x-x---
    x---x--
    -xxx---
    -------
    -xxx---
    x-x-xx-
    x-x-xx-
    ----x--
    -------

    -------
    x---x--
    -xxx---
    -------
    -xxxx--
    x----x-
    -------
    ---xxx-
    -------

    -------
    -xxx---
    -xxx---
    ----x--
    -xxxx--
    -xxxx--
    -----x-
    ----x--
    ----x--
    -------

    --x----
    -x-x---
    -x--x--
    ----x--
    -x---x-
    -x---x-
    --x--x-
    ----xx-
    -------

    --x-----
    -x-x----
    --xxx---
    ----xx--
    ----xx--
    -xx-xxx-
    -----xx-
    ----xx--
    --------

    --x-----
    -x--x---
    --x--x--
    --------
    --------
    ---x----
    ---x----
    ----xxx-
    --------

    -------
    -xxx---
    -------
    -------
    -------
    -------
    ---x-x-
    ----xx-
    -----x-
    -------

    --x-----
    --x-----
    --x-----
    --------
    --------
    --------
    -----x--
    -----xx-
    ----xx--
    --------

    --------
    -xxx----
    --------
    --------
    --------
    --------
    -----xx-
    ------x-
    ----xxx-
    --------

    --x------
    --x------
    --x------
    ---------
    ---------
    ---------
    -----xx--
    ----x--x-
    -----xx--
    -----x---
    ---------

    ---------
    -xxx-----
    ---------
    ---------
    ---------
    ---------
    -----xx--
    ----x--x-
    ----xxx--
    -----xx--
    ---------

    --x------
    --x------
    --x------
    ---------
    ---------
    ---------
    -----xx--
    ----x--x-
    ----x--x-
    ----x-x--
    ---------

    ---------
    -xxx-----
    ---------
    ---------
    ---------
    ---------
    -----xx--
    ----x--x-
    ---xx-xx-
    -----x---
    ---------

    --x------
    --x------
    --x------
    ---------
    ---------
    ---------
    -----xx--
    ---xx--x-
    ---xx-xx-
    ----xxx--
    ---------

    ---------
    -xxx-----
    ---------
    ---------
    ---------
    ---------
    ----xxx--
    ---x---x-
    -------x-
    ---xx-xx-
    -----x---
    ---------

    --x-------
    --x-------
    --x-------
    ----------
    ----------
    -----x----
    ----xxx---
    ----xx-x--
    ---xx--xx-
    ----xxxx--
    ----xxx---
    ----------

    ----------
    -xxx------
    ----------
    ----------
    ----------
    ----xxx---
    ----------
    -------xx-
    ---x----x-
    --------x-
    ----x--x--
    -----x----
    ----------

    --x--------
    --x--------
    --x--------
    -----------
    -----x-----
    -----x-----
    -----xxx---
    -------xx--
    --------xx-
    -------xx--
    -----------

    -----------
    -xxx-------
    -----------
    -----------
    -----------
    ----xx-----
    -----x-xx--
    ---------x-
    ---------x-
    -------xxx-
    -----------

    --x---------
    --x---------
    --x---------
    ------------
    ------------
    ----xxx-----
    ----xxx-x---
    ---------x--
    ---------xx-
    --------xx--
    --------x---
    ------------

    ------------
    -xxx--------
    ------------
    ------------
    -----x------
    ----x-xx----
    ----x-xx----
    -----x--xxx-
    ----------x-
    --------x-x-
    --------xx--
    ------------

    --x----------
    --x----------
    --x----------
    -------------
    -----xx------
    ----x--x-----
    ----x----x---
    -----xxxxxx--
    --------x-xx-
    --------x-x--
    --------xx---
    -------------

    -------------
    -xxx---------
    -------------
    -------------
    -----xx------
    ----x-x------
    ----x----xx--
    -----xxx---x-
    ------x----x-
    -------xx-xx-
    --------xx---
    -------------

    --x-----------
    --x-----------
    --x-----------
    --------------
    -----xx-------
    ----x-x-------
    ----x--x--x---
    -----xxx---x--
    -----x--x--xx-
    -------xx-xx--
    -------xxxx---
    --------------

