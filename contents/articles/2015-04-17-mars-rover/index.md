---
title: Mars Rover Problem
author: Liam McLennan
date: 2015-04-17 20:32
template: article.jade
---

<p><img src="curiosity.jpg" style="width:710px;" alt="curiosity rover"/></p>

Mars Rovers is a simple programming problem with just enough complexity to be interesting and to provide sufficient challenge to experiment with some different strategies. The problem, as stolen from [here](https://code.google.com/p/marsrovertechchallenge/), is:

```
A squad of robotic rovers are to be landed by NASA on a plateau on Mars.

This plateau, which is curiously rectangular, must be navigated by the rovers so that their on board cameras can get a complete view of the surrounding terrain to send back to Earth.

A rover's position is represented by a combination of an x and y co-ordinates and a letter representing one of the four cardinal compass points. The plateau is divided up into a grid to simplify navigation. An example position might be 0, 0, N, which means the rover is in the bottom left corner and facing North.

In order to control a rover, NASA sends a simple string of letters. The possible letters are 'L', 'R' and 'M'. 'L' and 'R' makes the rover spin 90 degrees left or right respectively, without moving from its current spot.

'M' means move forward one grid point, and maintain the same heading.

Assume that the square directly North from (x, y) is (x, y+1).

Input:

The first line of input is the upper-right coordinates of the plateau, the lower-left coordinates are assumed to be 0,0.

The rest of the input is information pertaining to the rovers that have been deployed. Each rover has two lines of input. The first line gives the rover's position, and the second line is a series of instructions telling the rover how to explore the plateau.

The position is made up of two integers and a letter separated by spaces, corresponding to the x and y co-ordinates and the rover's orientation.

Each rover will be finished sequentially, which means that the second rover won't start to move until the first one has finished moving.

Output:

The output for each rover should be its final co-ordinates and heading.

Test Input:

5 5
1 2 N
LMLMLMLMM
3 3 E
MMRMMRMRRM

Expected Output:

1 3 N
5 1 E
```

The problem requires two things: a parser from the input to some useful data structure, and something to process that data structure. Immediately we are confronted by a fundamental difference between OO (at least as I see it done) and functional style.

The OO Programmer
---------

The OO programmer sees a `MarsRoverController` or similar abstraction for coordinating this problem. The `MarsRover` becomes a class having a `position` and a `direction`. The coordinating object depends on both the parser and the `MarsRover`. It firstly asks the parser to parse the input, then asks the `MarsRover` to process the movements. An even uglier implementation would have the `MarsRover` doing the coordination responsibility.

The Functional Programmer
----

To the functional programmer these are all just functions. A function to do the parsing and a function to process the data structure. In my implementation the functions are `parseInitialization` and `play`. My REPL usage looks like this:

```
> parseInitialization testInput |> play;;
val it : Rover list = [{x = 1;
                        y = 3;
                        direction = N;}; {x = 5;
                                          y = 1;
                                          direction = E;}]
```

Implementing the Action State Change
-----

Somewhere we have to process actions. The possible actions are turn left, turn right, or move forward. Processing one of this actions changes the state of the rover.

Let us first focus on the turning actions. There are different ways to implement this. My first attempt was a simple enumeration of the possibilities:

```
let turn direction action =
    if action = M then failwith "M is not a turn action"
    match (direction,action) with
        | (N,L) -> W
        | (N,R) -> E
        | (E,L) -> N
        | (E,R) -> S
        | (S,L) -> E
        | (S,R) -> W
        | (W,L) -> S
        | (W,R) -> N
```

I like the simplicity of this approach. The required guard to check for a move action was a strong indication that I had modelled the types incorrectly. If it feels wrong, it probably is.

As an experiment I then tried an implementation that understood the directions as angle measurements. This has the advantage that it could be extended to more realistic systems, but it is more complicated:

```
let turnAlt direction action =
    let directionAngles = [(N,0);(E,90);(S,180);(W,270)]
    let angle = (List.find (fun (d,a) -> d = direction) directionAngles) |> snd
    let angleToDirection angle =
        List.find (fun (d,a) -> a = angle) directionAngles |> fst
    let turnedAngle =
        match action with
        | L -> angle - 90
        | R -> angle + 90
        | M -> failwith "M is not a turn action"
    angleToDirection   (if turnedAngle < 0 then
                            turnedAngle + 360
                        elif turnedAngle = 360 then
                            0
                        else
                            turnedAngle)
```

Parsing
----

To keep things simple I started with simple string splitting to parse the input. For the record, it looks like:

```
let parseDirection = function
    | "N" -> N
    | "E" -> E
    | "S" -> S
    | "W" -> W
    | d -> failwith ("Unknown direction " + d)

let parseRover (line:string) =
    let lineParts = line.Split(' ')
    {x = lineParts.[0] |> int; y = lineParts.[1] |> int; direction = parseDirection lineParts.[2]}

let parseActions (line:string) =
    [
        for c in line do
            yield match c with
                    | 'L' -> L
                    | 'R' -> R
                    | 'M' -> M
                    | other -> failwith ("Unknown action " + (string other))
    ]

let parseInstructions lines : Instruction list =
    [
        for i in [0..(Array.length lines)/2 - 1] do
            let roverLine = lines.[2*i]
            let actionLine = lines.[2*i+1]
            yield {rover= parseRover roverLine; actions = parseActions actionLine}
    ]

let parseInitialization (input:string) =
    let lines = input.Split('\n')
    let topRightLine = lines.[0]
    {
        upperRightX = lines.[0].Split(' ').[0] |> int; upperRightY = lines.[0].Split(' ').[1] |> int;
        instructions = parseInstructions lines.[1..]
    }
```

Parsing with Fparsec
---

Then, for fun, I deleted my implementation and replaced it with [fparsec](http://www.quanttec.com/fparsec/) parser combinators.

Firstly, we need to parse the first line, giving the top right coordinates, to a point:

```
type Point = {x:int; y:int}
let ptopright = sepBy pint32 (pchar ' ') |>> (fun [x;y] -> {x=x;y=y})
```

usage:

```
> run ptopright "5 6"
val it : ParserResult<Point,unit> = Success: {x = 5;
 y = 6;}
```

Next, we need to parse a direction. This is an example of parsing alternatives:

```
type Direction = N | E | S | W
let pdirection = (charReturn 'N' N) <|> (charReturn 'E' E) <|> (charReturn 'S' S) <|> (charReturn 'W' W)
```

Now we can parse the second line of input, the rover's state. Here we run 5 parsers (x, space, y, space, direction) and pipe the results into a function that produces the type we want (`Rover`):

```
type Rover = {location:Point; direction:Direction}
// 1 2 N
let proverstate =
    pipe3
        (pint32 .>> (pchar ' '))
        (pint32 .>> (pchar ' '))
        pdirection
        (fun x y direction -> {location = {x=x;y=y}; direction = direction})
```

Next - a way to parse actions - turn left, turn right or move forward:

```
type Turn = L | R
type Action =
    | Turn of  Turn
    | M
let pturn = (charReturn 'L' (Turn L)) <|> (charReturn 'R' (Turn R))
let paction = pturn <|> (charReturn 'M' M)
```

Actions never occur by themself. We want to parse a sequence of them, so:

```
// LMMLMRM
let pactions = many (pturn <|> (charReturn 'M' M))
```

We want to parse pairs of rover + actions so:

```
type Instruction = {rover:Rover; actions:Action list}

// 1 2 N
// LMLMLMLMM
let pinstruction =
  pipe2
    (proverstate .>> newline)
    pactions
    (fun state actions -> {rover=state; actions=actions})
```

Finally, we connect all of our parsers together:

```
let pinit =
    pipe2
        (ptopright .>> newline)
        (many pinstruction)
        (fun topright instructions ->
            {upperRight = topright; instructions = instructions})
```

The Completed Mars Rover Solution
---

```
#r @"FParsec.1.0.1\lib\net40-client\FParsecCS.dll"
#r @"FParsec.1.0.1\lib\net40-client\FParsec.dll"

open FParsec

type Direction = N | E | S | W
type Point = {x:int; y:int}
type Rover = {location:Point; direction:Direction}
type Turn = L | R
type Action =
    | Turn of  Turn
    | M
type Instruction = {rover:Rover; actions:Action list}
type Initialization = { upperRight: Point; instructions: Instruction list}

let pdirection = (charReturn 'N' N) <|> (charReturn 'E' E) <|> (charReturn 'S' S) <|> (charReturn 'W' W)
let ptopright = sepBy pint32 (pchar ' ') |>> (fun [x;y] -> {x=x;y=y})

// 1 2 N
let proverstate =
    pipe3
        (pint32 .>> (pchar ' '))
        (pint32 .>> (pchar ' '))
        pdirection
        (fun x y direction -> {location = {x=x;y=y}; direction = direction})

let pturn = (charReturn 'L' (Turn L)) <|> (charReturn 'R' (Turn R))
let pactions = (many (pturn <|> (charReturn 'M' M))) .>> (skipChar '\n' <|> preturn ())

// 1 2 N
// LMLMLMLMM
let pinstruction =
    pipe2
        (proverstate .>> newline)
        pactions
        (fun state actions -> {rover=state; actions=actions})

let pinit =
    pipe2
        (ptopright .>> newline)
        (many pinstruction)
        (fun topright instructions ->
            {upperRight = topright; instructions = instructions})

let testInput = "5 5
1 2 N
LMLMLMLMM
3 3 E
MMRMMRMRRM"

let extectedOutput = "1 3 N
5 1 E"

let play (init:Initialization) =
    let playInstruction (instruction:Instruction) =
        let turn direction action =
            match (direction,action) with
                | (N,L) -> W
                | (N,R) -> E
                | (E,L) -> N
                | (E,R) -> S
                | (S,L) -> E
                | (S,R) -> W
                | (W,L) -> S
                | (W,R) -> N

        let applyAction (rover:Rover) = function
                | Turn t -> {rover with direction = turn rover.direction t}
                | M ->
                    let moved =
                        { rover with location = match rover.direction with
                                                | N -> {rover.location with y = rover.location.y+1}
                                                | E -> {rover.location with x = rover.location.x+1}
                                                | S -> {rover.location with y = rover.location.y-1}
                                                | W -> {rover.location with x = rover.location.x-1}
                        }
                    if moved.location.x < 0
                        || moved.location.y < 0
                        || moved.location.x > init.upperRight.x
                        || moved.location.y > init.upperRight.y then
                        rover
                    else
                        moved
        List.fold applyAction instruction.rover instruction.actions
    List.map playInstruction init.instructions

match run pinit testInput with
    | Success(result, _, _)   -> play result
    | Failure(errorMsg, _, _) -> printfn "Failed to parse: %s" errorMsg; []
```
