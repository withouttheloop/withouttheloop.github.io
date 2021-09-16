---
title: F# String Calculator
author: Liam McLennan
date: 2013-11-07 20:32
template: article.jade
---

I've done the [string calculator kata](http://osherove.com/tdd-kata-1/) before, most recently [in Haskell](http://withouttheloop.com/articles/2013-05-31-stringcalckata/). Here it is again in F#:

    let lToS (l:char list) = System.String.Concat(Array.ofList(l))
     
    let sumString =
        let parse (l:string) =
            let guessDelimeter (l:string) =
                match Seq.toList l with
                    | '/'::'/'::c::'\n'::_ -> c
                    | _ -> Seq.find (fun i -> i = ',' || i = '\n') l
            let valuePart = (fun (s:string) ->
                match Seq.toList s with
                    | '/'::'/'::c::'\n'::r -> r
                    | v -> v) >> lToS
            Array.map int ((valuePart l).Split(guessDelimeter l))
        (parse >> Array.fold (fun acc i -> acc + i ) 0)

     sumString "3,9,11" |> printfn "%d"
     sumString "3\n9\n11" |> printfn "%d"
     sumString "//;\n3;9;11" |> printfn "%d"


