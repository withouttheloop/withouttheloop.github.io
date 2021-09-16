---
title: Prometheus - Linking F# Records to Typescript Classes
author: Liam McLennan
date: 2014-09-20 20:32
template: article.jade
---

I would like to program in a way that is as strict, unambiguous and correct as possible. I want the compiler to assist as much as possible, for new development and for refactoring. SPJ has described a compiler as a, "weak theorem prover" and I want my theorems weakly proven. But there is a problem. A lot of my work is in the browser where JavaScript is the only option, and JavaScript does not meet any of my stated requirements. Fortunately, there is a neat little project called [TypeScript](http://www.typescriptlang.org/) that adds static type checking to JavaScript. 

For C# solutions there is a tool called [netjs](https://github.com/praeclarum/Netjs) that can compile C# code to TypeScript and thence to JavaScript. I tested it on a simple C# solution with success, however, it does not work for F# projects. 

For F# web projects my minimum requirement is a way to synchronize F# records with TypeScript classes, so that I may share equivalent type defininitions between my F# server application and TypeScript/JavaScript client application. I'm not a big fan of classes in F#, and neither discriminated unions nor tuples have an obvious mapping to JavaScript so for now I am content to stick to F# records.

[Prometheus](https://github.com/liammclennan/Prometheus) is a pre-alpha project (most of mine stay that way) that converts an F# assembly to a TypeScript file containing class equivalents of all the records in the F# assembly. By running prometheus immediately prior to your normal compilation step the sequence is something like:

<blockquote>
    Prometheus generates typescript definitions -> F# compilation -> TypeScript compilation
</blockquote>  

The flaw here is that sometimes you may have to build twice to get everything synced properly but for now that is a 1st world problem. The fix is to move the Prometheus compilation between the F# compilation and the TypeScript compilation. 

Give an F# assembly containing the following module:

	namespace MyModels

    	type Address = { number: int; street: string }
    	type Person = { name: string; age: int; address: Address }

Prometheus produces the following valid TypeScript file:

	module MyModels { 
	    export class Address {
	        constructor(public number: number, public street: string) {}
	    }
	}

	module MyModels { 
	    export class Person {
	        constructor(public name: string, public age: number, public address: Address) {}
	    }
	}

Now if my server and client side model definitions fall out of sync it will be a glorious compilation error. 