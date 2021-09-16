---
title: Functional Game of Life part 2 (Haskell)
author: Liam McLennan
date: 2013-05-08 20:32
template: article.jade
---

<style>
.cell {
    border: 2px solid black;
    width: 18px;
    height: 18px;
    position: absolute;background-color: #eee;
}
.on {
    background-color: #333;
}
</style>

> DISCLAIMER: I don't know Haskell at all. I know the code in this article is not idiomatic and I look forward to receiving constructive criticism.

In [part 1 of this series](../2013-05-06-functional-game-of-life/) I began to implement the game of life cellular automaton in C# using the functional programming style (at least as I know it). Part 1 defined the data model and implemented a function to render the state of the system to HTML. 

For this part 2 of the series I port my C# solution to Haskell. 

Firstly, the data structure. As with the C# version I model the game of life as a list of living cells. The definition of a `cell` is:

    data Cell = Cell {
      x       :: Int,
      y       :: Int
    } deriving (Show)

where x and y are functions that return the cartesian coordinates of the cell. It is the same as the C# version, except that this `Cell` type is immutable.

Having defined my `Cell` type I then defined a set of functions to do useful things with a `Cell` or a list of cells. `renderLocation` is a function that converts a point (xv,yv) and a boolean (indicating if the cell at that location is living) into its HTML string.

    renderLocation :: Int -> Int -> Bool -> String
    renderLocation xv yv on = printf 
                              "<div class=\"cell %s\" style=\"left: %dpx; top: %dpx;\">&nbsp;</div>\n" 
                              (if on then "on" else "") (20 * (xv+1)) (20 * (yv+1))

The first line is a type definition for the function. Its C# equivalent would be `Func<int,int,bool,string>`.

To make `renderLocation` useful I need a function to determine if a given location contains a living cell:

    isOn :: [Cell] -> Int -> Int -> Bool
    isOn cs xv yv = any cellMatch cs
      where cellMatch c = x c == xv && y c == yv

This roughly translates to the C# `cs.Any(c => x(c) == xv && y(c) == yv)`.

The other helper functions required calculate the largest X and Y values to determine the size of the system:

    largestX :: [Cell] -> Int
    largestX cs = maximum $ map x cs

    largestY :: [Cell] -> Int
    largestY cs = maximum $ map y cs

`largestX` translated to C# is:

    public int largestX(Cell[] cs) {
      return cs.Select(c => x(c)).Max();
    }

Next I created three living cells, at coordinates (1,2), (3,5) and (14,13):

    cells = [Cell 1 2, Cell 3 5, Cell 14 13]

Now I can create a `render` function that produces the HTML representation of a system:

    render :: [Cell] -> String
    render cs = concat [renderLocation x y (isOn cs x y) | x <- [0..xBound], y <- [0..yBound]]
      where xBound = largestX cs
            yBound = largestY cs

The expression `[renderLocation x y (isOn cs x y) | x <- [0..xBound], y <- [0..yBound]]` is a list comprehension. The part after the pipe `x <- [0..xBound], y <- [0..yBound]` produces the cartesian product of all possible x values (`[0..xBound]`) and all possible y values (`[0..yBound]`). For each point the `renderLocation` function is called. The results are collected into a list of strings which is then flattened to a single string by the `concat` function.

This is equivalent to the C# version:

    var results = from x in Enumerable.Range(0, largestX+1)
                  from y in Enumerable.Range(0, largestY+1)
                  select buildCell(x, y, IsOn(cells, x, y));

Now to get the program to actually do something I define a list of cells and then use the programs `main` function to render my list of cells and write the result to stdout.

    cells = [Cell 1 2, Cell 3 5, Cell 14 13]

    main = (putStrLn . render) cells

The output of this program is:

<div style="clear:both;height: 400px;">
<div style="position:relative">
  <div style="left: 20px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 20px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 60px;" class="cell on">&nbsp;</div>
  <div style="left: 40px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 40px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 60px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 120px;" class="cell on">&nbsp;</div>
  <div style="left: 80px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 80px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 100px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 120px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 140px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 160px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 180px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 200px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 220px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 240px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 260px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 280px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 300px; top: 280px;" class="cell on">&nbsp;</div>
  <div style="left: 300px; top: 300px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 20px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 40px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 60px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 80px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 100px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 120px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 140px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 160px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 180px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 200px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 220px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 240px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 260px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 280px;" class="cell ">&nbsp;</div>
  <div style="left: 320px; top: 300px;" class="cell ">&nbsp;</div>
</div>
</div>

Here is the full program:

    import Text.Printf
    import Data.List

    data Cell = Cell {
        x       :: Int,
        y       :: Int
      } deriving (Show)

    renderLocation :: Int -> Int -> Bool -> String
    renderLocation xv yv on = printf 
                         "<div class=\"cell %s\" style=\"left: %dpx; top: %dpx;\">&nbsp;</div>\n" 
                         (if on then "on" else "") (20 * (xv+1)) (20 * (yv+1))

    isOn :: [Cell] -> Int -> Int -> Bool
    isOn cs xv yv = any cellMatch cs
      where cellMatch c = x c == xv && y c == yv

    largestX :: [Cell] -> Int
    largestX cs = maximum $ map x cs

    largestY :: [Cell] -> Int
    largestY cs = maximum $ map y cs

    render :: [Cell] -> String
    render cs = concat [renderLocation x y (isOn cs x y) | x <- [0..xBound], y <- [0..yBound]]
      where xBound = largestX cs
            yBound = largestY cs

    --cells = [Cell 1 2, Cell 3 5, Cell 14 13]
    --new Cell(1,0), new Cell(0,1), new Cell(2,1), new Cell(7,16)
    cells = [Cell 1 0, Cell 0 1, Cell 2 1, Cell 7 16]

    main = (putStrLn . render) cells

It is a bit longer than the C# version, but that is because I have used more explicit functions and specified more explicit types.
