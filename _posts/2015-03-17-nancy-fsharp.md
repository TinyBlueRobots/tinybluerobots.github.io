---
layout: post
title:  "NancyFx |> F#"
date:   2015-03-17 15:45:22
categories: fsharp
---
So you want to build a website and, since you are a god amongst men, you want to use F#. Good, then I don't have to tell you how excellent [Nancy](http://nancyfx.org/) is, however, if you've used it with F# you’ve probably winced a little:

    type Module() as this =
      inherit NancyModule()

      do
        this.Get.["/"] <- fun p -> this.View.["Home"] |> box

This upsets me, so let's start again with what we'd like to see:

    let get() = View "Home"

And we just need a Response type:

    type Response =
        | View of string

Easy. That view name is a source of concern, one could easily mistype that. Thankfully we have the FSharp.Management file type provider to aid us:

    type Views = RelativePath< ".\\Views", watch=true >

Let's try again:

    let get() = View Views.``Home.cshtml``

Nice. Next we need to handle view models, which are optional, so let's update the Response:

    type Response<'a> =
      | View of string * 'a option

    let get() = View(Views.``Home.cshtml``, None)

Next, those dynamic parameters. F# has the ? operator for just this job, and we can clean up those dirty nulls:

    let (?) (p : obj) prop = 
      let ddv = (p :?> DynamicDictionary).[prop] :?> DynamicDictionaryValue
      match ddv.HasValue with
      | false -> None
      | _ -> 
        try 
          ddv.Value |> unbox<'a> |> Some
        with :? InvalidCastException -> None

This is much more pleasing, and we can build up a testable library of functions without even thinking too much about Nancy. Here are some example [```Modules```](https://github.com/TinyBlueRobots/NancyFs/blob/master/src/NancyFs/Modules/Modules.fs).

Next we just have to wire them up. Fear not, I've written the boiler plate for you so we just have to do a bit of composition:

    type Routes() as this = 
      inherit NancyFsModule()
      do 
        (fun _ -> Home.get()) |> this.CreateRoute GET "/"
        (fun _ -> this.Bind<NameModel>() |> Home.post) |> this.CreateRoute POST "/"
        (fun _ -> this.Request.Query?name |> About.get) |> this.CreateRoute GET "/about"
        StaticFile.get |> this.CreateRoute GET "/{file}"
        (fun p -> p?redirect |> Redirect.get) |> this.CreateRoute GET "/redirect/{redirect}"

Async? Got you covered there too, just return an async from your function and use:

    this.CreateAsyncRoute GET "/route"

In this case the function will also require a cancellationToken parameter.

Before you embark on this adventure, ensure you have F# Power Tools installed for folder creation and manipulation, and you've installed [these registry extensions](http://bloggemdano.blogspot.co.uk/2013/11/adding-new-items-to-pure-f-aspnet.html) so you can create files. Now clone this and get going:

[https://github.com/tinybluerobots/nancyfs](https://github.com/tinybluerobots/nancyfs)

Have a look at [```Nancy.fs```](https://github.com/TinyBlueRobots/NancyFs/blob/master/src/NancyFs/Modules/Nancy.fs) and you'll see the ```Response``` type with a few implementations; simply adjust to your needs.
```NancyFsModule``` is the carpet under which we are sweeping all the unpleasantness, and the  ```Nancify``` method is where we convert the ```Response``` into something Nancy can consume.

We're using this in production, and with FsPickler and fszmq to talk to the back end we get the benefit of F# everywhere.

Let me know what you think and if you can improve on what we've done.