---
title: Nokia Ringtone Composer Emulator
author: Liam McLennan
date: 2015-10-28 20:32
template: article.jade
---

I've been working on a Pluralsight F# course for some time. The sample project I chose is to build a web-based emulator of the classic Nokia 3310 composer application that allowed Nokia owners to program their own ringtones. I have previously written about [building the tone generator](/articles/2014-10-29-fsharp-signal-generator/) and [parsing the ringtone syntax with FParsec](/articles/2014-10-30-building-a-parser-with-fparsec/).

The final step was to wrap the composer application in a simple web page, and so

> my [Nokia Composer Emulator](http://nokiacomposer.azurewebsites.net/) is now online!

Using FSharpX to make it easier to work with Choices
=============

The composer module returns a [`Choice<MemoryStream,string>`](https://msdn.microsoft.com/en-us/library/ee353439.aspx) that represents either a wave file or a string error message. `Choice` is a type with two possible forms, a choice value is either a `Choice1Of2 someMemoryStream` or `Choice2Of2 "error message"`.

To work with a choice we pattern match:

```
    match aChoiceValue with
        | Choice1Of2 stream -> doSomethingWith stream
        | Choice2Of2 error -> printfn "Error: %s" error
```

I got sick of constantly pattern matching the choice, doing some calculation and repacking a choice, so I looked to [FSharpx](http://fsprojects.github.io/FSharpx.Extras/reference/fsharpx-functional-choice.html) for some help. For my scenario I want do something with the successful value if there is one and pass straight through if there has been an error. FSharpX provides [`Choice.map`](https://github.com/fsprojects/FSharpx.Extras/blob/master/src/FSharpx.Extras/ComputationExpressions/Monad.fs#L751-751) for this purpose.

Code goes from

```
let assembleToPackedStream (score:string) =
    match parse score with
        | Choice1Of2 tokens -> Choice1Of2 <| pack(assemble tokens)
        | Choice2Of2 error -> Choice2Of2 error
```

to

```
let assembleToPackedStream (score:string) =
    Choice.map (fun tokens -> pack(assemble tokens)) (parse score)

```

If, when processing the success value, I wanted to be able to produce an error I could have used [`bind`](https://msdn.microsoft.com/en-us/library/ee353439.aspx) instead of `map`:

```
let assembleToPackedStream (score:string) =
    Choice.bind (fun tokens ->
                    if thereIsAnError then
                        Choice2Of2 "error message goes here"
                    else
                        Choice1Of2 (pack(assemble tokens)))
        (parse score)
```

More commonly people use the `>>=` infix operator for the bind operation.

```
let assembleToPackedStream (score:string) =
    (parse score) >>= (fun tokens ->
                        if thereIsAnError then
                            Choice2Of2 "error message goes here"
                        else
                            Choice1Of2 (pack(assemble tokens)))
```


F# Web Application
============

For the web application I use the excellent [asp.net mvc 5 and web api 2 F# project template](http://bloggemdano.blogspot.com.au/2013/12/a-new-f-aspnet-mvc-5-and-web-api-2.html).

The action I added to handle the composition post converts an error result to an exception. If the composition was created successfully then the `Content-Dispostion` header is set and the wave file returned.

```
[<HttpPost>]
   member this.Produce(score:string) =
       match Assembler.assembleToPackedStream score with
           | Choice1Of2 ms ->
               this.Response.AppendHeader(
                   "Content-Disposition",
                   (ContentDisposition(FileName = "sickringtone.wav", Inline = false)).ToString())
               ms.Position <- 0L
               this.File(ms, "audio/x-wav")
           | Choice2Of2 err -> failwith err
```

In the past I have also used the [nancy F# templates](http://bloggemdano.blogspot.com.au/2013/12/a-few-other-template-additions-and.html) which are good.
