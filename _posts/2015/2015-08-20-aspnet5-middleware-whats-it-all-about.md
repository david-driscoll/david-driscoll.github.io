---
layout: post
title: "Asp.Net 5 Middleware... What's it all about?!?"
tags: aspnet aspnet5 aspnetvnext dnx middleware express expressjs connect connectjs
comments: true
---

The Asp.Net landscape is changing rapidly.  We have servers running on Linux, Mac and Windows.  This is something that until recently is unheard of for Microsoft, and it's **awesome**.  This also means that sometimes you need to get what you know, and learn the new way.  Http Modules and Handlers for example are going away, and they're being replaced with this new fangled thing called Middleware.  You might ask what is Middleware... we'll get to that.

_<sub>Warning the following information may cause the following emotions :anguished: :sweat: :rage: :cry: :fearful:, they will hopefully be followed by :relaxed: :smile:</sub>_

## Out with the old...
Today if you want to make some portable way of modifying requests into your Asp.Net application, you would have two choices, depending on your needs.  Http Modules, and Handlers, they each served a specific purpose, and were both equally useful.  In Asp.Net 5, they are **_GONE_**.

* Http Module [kb]

    Modules allowed you to tap into the Lifecycle of your requests, do things like add custom authentication or authorization, request redirecting, and other fun things.  Mastering Http Modules made you a god among of your peers, because lets be honest they aren't always the most straight forward.

    To hook in a module you had to either use a programmatic API that was undocumented and always made me feel a little dirty (`DynamicModuleUtility` I'm looking at you! ::) or you added a convoluted web.config entry.  Order mattered because any module that ran before you could redirect the request before you had a chance to deal with it (`FormsAuthenticationModule`... :imp:).

* Http Handler [kb]

    Handlers allowed you to materialize something from... well memory... or the database... or some other cache.  I've used them for emitting embedded resources (JavaScript, CSS, etc) or just making a dynamic response where I don't want the full weight of MVC.  All in all, they're very useful.

    To hook in a handler you had to use web.config... and don't try to register the same handler twice or IIS gets really really angry. :rage4:

Now don't worry, the sky is not falling!   You can still do the exact same things, just how you go about accomplishing that is _different_.

## In with the new...

So, to recap Modules and Handlers are ~~**GONE**~~.  You can still use them as part of an IIS deployment if you really wanted to, but Asp.Net doesn't just run in IIS, you can use [Kestrel](https://github.com/aspnet/KestrelHttpServer) on a Mac for example.

The replacement is thing called "Middleware".  Wikipedia is no help, it tells us Middleware is software that runs in the middle of something else... but that means basically everything from the kernel to drivers to the OS is a Middleware of some kind.

### So what is Middleware?
Middleware is essentially a chain of callbacks, or a `pipeline`.  The callbacks get the incoming request as input, they do there work, and then choose to continue down the pipeline or not.

> If you have ever looked at NodeJS, several libraries have popularized the use of Middleware, [Connect](https://github.com/senchalabs/connect) and [Express](http://expressjs.com/).  A lot of what Asp.Net 5 does, is very similar in concept to the NodeJS way.

Your callback can...

* add HTTP Headers
* modify the HTTP response
* authenticate the user
* log information about the request
* anything you can imagine

Order has importance, your Middleware will run in order it was added.  Your Middleware has the choice to continue, fork or stop.

* continue will call the next Middleware that was added.
  * This is great for things like autentication, or error logging for example.
* fork will call a different Middleware, and you can control when to follow the fork.
  * This allows you to do things like isolate a section of your application, based on the path and the user that's authenticated.
  *
* stop will not call the next Middleware and the request will return.

With the ability to control absolute ordering you are not longer tide to a specific set of application lifecycle of events.  You in fact get to build your own application lifecycle.  You can make both _good_ choices or _bad_ choices, but whats important is you get to make the _**choice**_.


### Why Middleware?
So you might ask, why Middleware?  Why not something else entirely?

**Performance Tax!**  You get to choose how to compose your application will the Middleware available to you.  A good way to look at it is as Lego blocks (perhaps Minecraft blocks if that's your thing!).  You can pick and choose how you put the blocks together to make something _awesome_.  The Asp.net team calls this `pay-for-play`, you only pay for what you're using.

In addition you can stop paying for a feature you'll never use.  If aren't making a social application you can _remove_ facebook, google and microsoft authentication.   If you don't need Mvc, you don't use MVC.  If you don't need Razor, you don't load Razor.  If all you're going to serve is lolcats, you can remove everything but the code you need to serve lolcats.


### Types of Middleware
As you saw earlier, there are several different kinds of Middleware.

#### Terminal Middleware
The first simplest is terminal Middleware, this Middleware does not have to call through to the next caller, you can simply act on the request.  Below is Hello World in Asp.Net 5.

{% gist david-driscoll/a2f1ea8e2ceed0f74da4 %}

This uses `app.Run` which is a terminal Middleware builder.  If you change it up and try to add another `app.Run` call it will do nothing.

{% gist david-driscoll/6c693ca86117bdf9b418 %}

#### Chaining Middleware
You can then chain Middleware off of each other, to produce a combined result at the end.

{% gist david-driscoll/8e5869d542f3daabb54b %}

#### Forking Middleware
Forking Middleware lets you make a decision, in this example you can choose to go one way when the condition is met.  In this example I'm just using an Arbitrary random number to produce a result.

{% gist david-driscoll//a53cfb45c9b2b1a82250 %}

### Out of the Box Middleware
Out of the box there is lots of Middleware that comes with Asp.Net 5.  

* [CORS](https://github.com/aspnet/CORS)
* [Diagnostics](https://github.com/aspnet/Diagnostics)
    * Includes Diagnostic Page, Error Pages (New yellow screen of death!), Runtime Information, Status Code Pages, Welcome Page, etc.
* [Localization](https://github.com/aspnet/Localization)
* [Mvc](https://github.com/aspnet/Mvc)
* [Security](https://github.com/aspnet/Security)
    * Includes cookie auth, facebook, google, micorosft, oauth, openid, twitter, etc.
* [StaticFiles](https://github.com/aspnet/StaticFiles)
    * Includes static files, directory browser, etc .
* [Session](https://github.com/aspnet/Session)
* [ResponseCaching](https://github.com/aspnet/ResponseCaching)
* [Routing](https://github.com/aspnet/Routing)

-----------


In essence Middleware makes your applications pipeline explicit, which makes it easier to understand how a particular application will behave given different incoming requests.   Gone are days where magic just happens, and with it, needing to understand the magic.


If you're still not quiet sure what Middleware is or how it is helpful, reach out to myself on twitter, or even touch base with the aspnet5 team via #aspnet5.


[kb]: https://support.microsoft.com/kb/307985 "Microsoft Knowledge base article on Http Modules and Handlers"
