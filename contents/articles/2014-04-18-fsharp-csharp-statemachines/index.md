---
title: State Machines in F# and C#
author: Liam McLennan
date: 2014-04-18 20:32
template: article.jade
---

A couple of months ago I wrote a [simple library for building state machines in C#](https://gist.github.com/liammclennan/8949046). It allows a programmer to attach a state machine to a type and control:

* the set of states that the type may have
* the transitions between those states

The first system that used the library contained functionality for managing the lifecycle of contracts. The contracts begin in a draft state, they may then move to an approved state, and finally they may enter the historical state. They cannot go from draft to historical, from approved to draft, from historical to draft etc. Using the state machine added a valuable invariant guarantee to the code. 

As much as my state machine library was a successful solution to a problem I can't help but realise that this problem simply doesn't exist in F#. F#'s type system is sufficiently advanced so that it can represent state machines as part of the type. This meets the stated goal of my C# state machine library. To use the previous contracts example, the F# type would look like this:


    type Particulars = { buyer: string; seller: string }

    type Contract = 
        private
        | Draft of Particulars
        | Approved of (Particulars * DateTime)
        | Historical of (Particulars * DateTime)
        
    let createDraft particulars = 
        Draft particulars
        
    let approve timestamp = function
        | Draft p -> Approved (p, timestamp)
        | _ -> failwith "Invalid transition"

    let retire timestamp = function 
        | Approved (p,t) -> Historical (p, timestamp)
        | _ -> failwith "Invalid transition"

    // usage
    let contract = createDraft { buyer = "Acme"; seller = "Initech" }
    let approved = approve DateTime.Now contract
    let retired = retire DateTime.Now approved
    retired |> printfn "%+A"

    // output = Historical ({buyer = "Acme"; seller = "Initech";}, 
    //                          18/04/2014 1:27:20 PM)

The type system models the relationship between Contract and each of the possible states (a Contract is a draft, or approved or historical) as well as the data attached to each state (a draft doesn't have a timestamp).