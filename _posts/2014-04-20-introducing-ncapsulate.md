---
layout: post
title: Introducing Ncapsulate
tags: ncapsulate node .net c# nuget
comments: true
---

![logo](https://raw.githubusercontent.com/Blacklite/NCapsulate/master/logo-large.png)
https://github.com/Blacklite/Ncapsulate

Ncapsulate is a NuGet wrapper for NodeJS and libraries of the Node Ecosystem.  Such tools currently include Bower, Gulp, Grunt, Npm, Karma, and support is easy to add for more.  Ncapsulate allows Front-end Developers to bring the Node Ecosystem into the .NET world without making your manager freak out at having to add "yet another tool" for everyone to install and work with.


## Why should I use Ncapsulate over just using Node by itself?
* My team already uses Node and Bower, should we use Ncapsulate?
It depends, if you work with a small team that is fine with using Node and related libraries, then there is no reason why you would need to use Node by itself.  Keep  in mind Ncapsulate also has MSBuild integration so you can execute Gulp tasks, dependency install, and more directly from your Project. With minimal effort.

* It is difficult for my team to adopt new tools that need to be installed, could we use Ncapsulate?
Yes!  Ncapsulate is portable and does not require any external dependency to be installed on Developer machines.  Ncapsulate is also completely compatible with Nuget Package Restore.


## Ncapsulate Use Cases


Take [Bower](bower) for example, with bower you can install any number of client side technologies from [Twitter Bootstrap](bootstrap), [Angular](angular) and [Font-Awesome](font-awesome).  With no native way in Visual Studio to utilize bower we must use Nuget for some of these dependencies.

In the NuGet ecosystem some libraries are updated often and regularly to keep in sync with Bower releases but this is not always the case.  In addition because of the nature of NuGet's content system, all of these libraries come with a bunch of assumptions in where the content get's  placed or used.

Ncapsulate allows us to use [Bower](bower) as we would normally operate through the command line, but instead we have nicer tight integration with MSBuild itself, remember csproj files are just MSBuild xml under the covers.




This also opens the door for using [Less](less) as part of our build process.  Yes there are tools that developers can use such as Web Essentials.  And I think Web Essetentials is awesome, I use it every day but the problem is the Editor can compile these things for us, but the build server cannot.  If the Developer doesn't have the extension installed, or maybe the make a manual change to the css instead of the less.  These are easy mistakes to make, but they can cause signifcant issues down the road for the next developer.


Want need configurable task runner?  Combine [Gulp](gulp) or [Grunt](grunt) with MSBuild and you can run the advanced tasks like r.js in a NodeJs environment, and then package it all up with MSBuild and deploy to Azure!


## How to use Ncapsulate?
Pretty simple, using Nuget of course!

![Nuget Search](/public/introducing-ncapsulate/2014-04-20%2000_01_41-ncapsulate.example%20-%20manage%20nuget%20packages.png)
Install like any other Nuget Dependendency!

<span style="float:left">
    <img src="/public/introducing-ncapsulate/2014-04-20 00_12_43-master-NCapsulate.Example.png" />
</span>
Once installed you'll see some new files in your solution, these can be excluded from the solution if you have no need for them.  Keep in mind the .cmd files must still be checked into your source control solution for Ncapsulate's tooling to work correctly.

The .targets are completely optional but they are Ncapsulates default integration point with the specific node tool.  For Npm it will install and update your dependencies, same for Bower.  For Gulp and Grunt you can link certian tasks to a specific phase of the build.  You can compile your JavaScript `BeforeBuild` and run your Unit Tests with Karma `AfterBuild`.

And that's it, you're ready to get started!

<p>&nbsp;</p><p>&nbsp;</p><p>&nbsp;</p><p>&nbsp;</p><p>&nbsp;</p><p>&nbsp;</p>

## Can I contribute to Ncapsulate?
Absolutely, there are many many, many tools to be implemented with Ncapsulate for anyone to use.  I picked the top 4 that came to mind (my mind specificly) there are certianly a ton more, such as TSLint for linting typescript files, JSHint for linting JSFiles.  These can all be done through Grunt or Gulp as well.  The goal if Ncapsulate is to be the wrapper for the tools, and nothing more.


[grunt]: http://gruntjs.com
[gulp]: http://gulpjs.com/
[less]: http://lesscss.org/
[bower]: http://bower.io/
[bootstrap]: http://getbootstrap.com
[angular]: https://angularjs.org/
[font-awesome]: http://fortawesome.github.io/Font-Awesome/
