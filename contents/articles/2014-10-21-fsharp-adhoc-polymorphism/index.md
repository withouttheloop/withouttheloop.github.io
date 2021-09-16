---
title: Ad-hoc Polymorphism in F# (how to survive without Type Classes)
author: Liam McLennan
date: 2014-10-21 20:32
template: article.jade
---

A recurring source of confusion for me when reading about and writing F# has been ad-hoc polymorphism, and it seems that I am not the only one:

<blockquote>
"F# does not have support for type-classes, or for multi-methods, or anything in between and while you can do certain tricks to achieve such an effect, like passing around a global virtual table, truth is F#'s ad-hoc polymorphic capabilities really suck."
</blockquote>
[Martin Trojer](http://martinsprogrammingblog.blogspot.com.au/2012/10/the-future-of-net-lies-in-mono-future.html)

[Ad-hoc polymorphism](http://en.wikipedia.org/wiki/Ad_hoc_polymorphism) is the programming language feature that allows a function to work with arguments of varying types. The example given by wikipedia is:

	1 + 2 = 3
	3.14 + 0.0015 = 3.1415

The first line shows the addition of two integers. The second line shows the addition of two floating point numbers. For this to work the `(+)` operator must be a function that can operate on both integers and floats. This is an example of ad-hoc polymorphism. F#'s predecessor OCaml has almost no support for ad-hoc polymorphism, so the previous `(+)` example does not work in OCaml. `+, -, / and *` only work for integers and you need `+., -., /. and *.`	 for floating point operations. 

Where F# Becomes Difficult
--------------------

Consider the following example (borrowed from [http://blog.lab49.com/archives/2895](http://blog.lab49.com/archives/2895)):

	// this does not work :( 
	// (expression was expected to have type 
	// float but here has type int)
	let twice x = x + x

	twice 2.0 |> Console.WriteLine
	twice 2 |> Console.WriteLine

This code fails because the x parameter of the twice function is inferred to be of type `float`, because that is the type of the argument the first time that it is called. Type inference lets us get away with not explicitly specifying a type, but it still has to assign a single type. In this case it chooses float, so when the function is applied to an integer we get a compilation error. 


In F# the twice function above can fixed by means of the `inline` modifier. 

	// this version works!
	let inline twice x = x + x

	twice 2.0 |> Console.WriteLine
	twice 2 |> Console.WriteLine

Each call to the inline function is replaced by inline code with its own type inference, thus the compiler effectively converts the above to:

	(fun (x:float) -> x + x)(2.0) |> Console.WriteLine
	(fun (x:int) -> x + x)(2) |> Console.WriteLine

This is rather nice because we get the explicit type correctness of having no runtime type coercion, but without having to manually provide implementations for each type. In this way it is superior even to [Haskell's lauded type class feature](http://www.haskell.org/tutorial/classes.html). 

A More Realistic Example
---------------------

Imagine you want to define a function `show` that converts a value of type `A` or `B` to a string representation. My first attempt was:

	type A = { thing: int }
	type B = { label: string }
	 
	let show (a:A) =
		sprintf "%A" a
	 
	let show (b:B) =
		sprintf "%A" b
	 
	{ thing = 98 } |> show |> Console.WriteLine

This is not allowed. F# does not allow you to overload the `show` function in this way.

How About Using an Interface?
-------

Often the requirement for ad-hoc polymorphism can be achieved by using interfaces. F# programmers often talk about using a 'dictionary of operations' which is essentially the same thing.

The example above becomes:

	type IShow =
		abstract member show : unit -> string
	 
	type A =
		{ thing: int }
		interface IShow with
			member this.show() = sprintf "%A" this
	type B =
		{ label: string }
		interface IShow with
			member this.show() = sprintf "%A" this
	
	let show (x:IShow) =
		x.show()

	{ thing = 98 } |> show |> Console.WriteLine
	{ label = "Car" } |> show |> Console.WriteLine

This is OK, but a little clumsy. There are also some restrictions on implementing interfaces that may occassionally make this solution infeasible. 

F# allows type coercion to an interface for function arguments so we avoid having to explicitly upcast values to IShow (`({ thing =98 } :> IShow).show()`). 

Solution Using Static Member Overloading
------------

My favourite solution, at least so far, is to use static member overloading. This takes advantage of the fact that while top-level functions cannot be overloaded functions on types can.

	type A = { thing: int }
	type B = { label: string }
	 
	type ThingThatShows =
		static member show(x:A) = sprintf "%A" x
		static member show(x:B) = sprintf "%A" x
	
	{ thing = 98 } |> ThingThatShows.show |> Console.WriteLine
	{ label = "Car" } |> ThingThatShows.show |> Console.WriteLine

This nice part about this is that I am able to define two versions of `ThingThatShows.show`, one for type `A` and one for type `B`. The correct function is then applied based upon the type of the argument.

Solution Using Statically Resolved Type Constraints
-----------------------------------

F# has a somewhat unusual feature - statically resolved generic types - and these can be used in combination with member constraints and inline functions to implement ad-hoc polymorphism. 

	type A = { thing: int }
		with static member show a = sprintf "%A" a
	type B = { label: string }
		with static member show b = sprintf "%A" b

	let inline show (x:^t) =
		(^t: (static member show: ^t -> string) (x))

	{ thing = 98 } |> show |> Console.WriteLine
	{ label = "Car" } |> show |> Console.WriteLine

This is a lot like the solution using interfaces except:

* I did not have to define an interface.
* I did not have to explicitly implement the interface members.

What Does This Look Like in Fancy-pants Haskell?
-------------

Haskell does not have this ad-hoc polymorphism problem. It is obvious in Haskell that Type Classes are the correct solution to this challenge. The example repeated above when converted to a Type Class based Haskell solution is:

	-- define a type class for types
	-- that can be shown
	class Showable a where
		shw :: a -> String
 
	data A = A String
	data B = B Int
 
	-- add the types A and B 
	-- to the Showable type class
 	
	instance Showable A where
		shw (A s) = s
 
	instance Showable B where
		shw (B i) = show i

	putStrLn (shw (A "Car"))
