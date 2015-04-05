---
layout: post
title: Dependency Injection with DNX
tags: aspnet aspnet5 aspnetvnext dnx coreclr di
comments: true
---

Traditionally in AspNet applications Dependency Injection was fairly hard to pull off, and even harder to pull off properly.  In fact it usually felt a little like pulling teeth with a sledge hammer :hammer:.  Welcome to the new world with [DNX](https://github.com/aspnet/dnx).  Not only is .NET going [Cross-Platform](https://github.com/aspnet/home#os-xlinux) but Dependency Injection is now a first class citizen!


## What is Dependency Injection?
If you've lived under a rock, or in the past you heard the words "XML Configuration" and promptly ran away :scream: (I don't blame you) then you might not know what Dependency Injection is about.

[Wikipedia](http://en.wikipedia.org/wiki/Dependency_injection) tells us...
> In software engineering, dependency injection is a software design pattern that implements **inversion of control** for software libraries. Caller delegates to an external framework the control flow of discovering and importing a service or software module.

In a nut shell what it means is that instead of writing code like...

```csharp
public class MyController {
    private readonly DbContext _context;
    public MyController() {
        _context = new DbContext();
    }
}
```

You write code like...

```csharp
public class MyController {
    private readonly DbContext _context;
    public MyController(DbContext context) {
        _context = context;
    }
}
```

What [Inversion of Control](http://en.wikipedia.org/wiki/Inversion_of_control) means is that your class no longer needs to understand how to construct it's dependencies.  Instead the Container will hand you a list of dependencies for you to use.

At first you'll be scared :scream: or perhaps even angry :angry:.  There are advantages to entering this brave new world.  

* Unit testing becomes much much easier, when you can easily stub and mock and dependencies that you are injecting.
* Code sharing is much easier, you don't have to understand where, or how you get a `DbContext` you just ask the Container for one and it will serve it up for you.
* Code is decoupled, and you no longer have to understand the inner workings of a dependency.


There are also some disadvantages to this brave new world... :disappointed:

* Code is decoupled, so you no longer understand the inner workings of a dependency.
> But you said this was a advantage?  It is both a pro and a con in some circumstances.  Generally it is a bigger advantage than disadvantage.
* Potential unseen overhead during object creation.


## What Dependency Injection looks like today
It looked something a little like the following<sup>[[source](http://www.asp.net/mvc/overview/older-versions/hands-on-labs/aspnet-mvc-4-dependency-injection)]</sup>:

```csharp
public static class Bootstrapper {
    public static void Initialise() {
        var container = BuildUnityContainer();

        DependencyResolver.SetResolver(new Unity.Mvc3.UnityDependencyResolver(container));

        IDependencyResolver resolver = DependencyResolver.Current;

        IDependencyResolver newResolver = new Factories.UnityDependencyResolver(container, resolver);

        DependencyResolver.SetResolver(newResolver);
    }
    ...
}
```

You have to build the container, setup the static resolver using the container, and things get worse when you want inject services into an `IHttpModule` or `IHttpHandler`.  Basically it's a lot of hoops.  It was never really fun or intuitive.  In fact it wasn't part of the original design of AspNet, so it was tacked on after AspNet was built.


## What Dependency Injection looks like in DNX
If you do not already know DNX is the new execution environment, standing for ".NET Execution" or ".NET Cross Platform".  It is the system that loads (or compiles) your code and hosts it.

As part of this new environment Dependency Injection is included as a first class citizen.  Almost all of the internal code is constructed using a simple, no fancy pants :jeans:, lightweight :cloud: dependency injection container.  This means as soon as you start to write your code you can begin to use Dependency Injection.

### Configuring Services
As part of your application you are going to have to register your services.  This can be achieved by adding items to the `IServiceCollection`.

In this brave new world :earth_americas: you "Pay-for-Play".  That means that by default no services.  No mvc, No routing, etc.  To start using a new library, you have to specifically include those services.  When consuming library services, the current convention is to use an extension method on the `IServicesCollection` interface.

To use Mvc you have to add the Mvc services.  This applies to all sorts of other libraries as well, such as Entity Framework, etc.

```csharp
public void ConfigureServices(IServiceCollection services) {
    ...
    // Add Mvc services to the collection
    services.AddMvc();
    ...
}
```

To add your own service to the services collection you can call various other methods on it.

```csharp
public void ConfigureServices(IServiceCollection services) {
    services.AddTransient<IInterface, MyClass>();
}
```

This will add `IInterface` to the service collection.

#### Lifecycles
Every object has a finite lifetime.  Sometimes it lives for the entire life of your application, like a static singleton.  Sometimes you want a new item each and every time to work with.  Other times you want the same item for an http request.  Same can be done with Dependency Injection, you just have to tell the container what the intended lifetime is.

* Singleton<br />
  These services will be constructed once, and from that point forward any other request for this service will return the same instance.  It is like the traditional static singleton instance pattern.
* Instance<br />
  These services are registered with a specific instance, this is the singleton pattern, but the instance is constructed ahead of time.
* Scoped<br />
  These services will generally be constructed once per request, such as a common database context.
* Transient<br />
  These services will be constructed every time they are requested.  Just like calling `new Service()`

#### Factories
When configuring your services, you can also use a factory pattern to construct your dependencies.  Factory methods are not the most optimal way to register services, the preferred style would to create a proper Factory Interface that Injects all the services you need to construct your instances.

```csharp
public void ConfigureServices(IServiceCollection services) {
    services.AddTransient<IService>(x => new ServiceA(x.GetService<IServiceB>()));
}
```

#### Generic Types
With DNX you can register both open and closed generic types as well.  This allows you to do things like the Repository Pattern.  Inside AspNet5 there are examples of this being used for the new [Options](https://github.com/aspnet/Options) library.

```csharp
public void ConfigureServices(IServiceCollection services) {
    services.AddTransient(typeof(IRepository<>), typeof(RepositoryImpl<>));
    services.AddTransient(typeof(IConfigureOptions<MvcOptions>), typeof(MyMvcConfiguration));
}
```

### Special Types
There is also a class of special generic types, that will resolve intelligently for you as well.  Currently the only type that works for the simple container is `IEnumerable<>`.  This works much like the generic type resolution, but in this case it instead returns you an enumerable of *all* of the types that have been registered for that interface.  This is useful when you may have more than one implementation of a given interface.

```csharp
public class StrategyProvider {
    private readonly IEnumerable<Strategy> _strategies;
    public StrategyProvider(IEnumerable<Strategy> strategies) { _strategies; }

    public IStrategy GetStrategy(object instance) {
        return _strategies.First(x => x.IsFor(instance));
    }
}
```

Some other Dependency Injection containers also support injecting values of `Lazy<>` or `Func<>` but for the simple container built into DNX these are not supported out of the box.  You are able to use any other container with DNX based applications, such as [Autofac](http://alexmg.com/autofac-4-0-alpha-1-for-asp-net-5-0-beta-3/).


### Constructor Injection
By default constructor injection is used to define what services are required for a given implementation.

```csharp
public class MyService {
    public MyService(IServiceA serviceA, IServiceB serviceB, IServiceC serviceC) {
        ...
    }
}
```

In the above example, the container will return instances of `IServiceA`, `IServiceB` and `IServiceC`.  There is no knowledge of the lifecycle by `MyService`.


### Injectables
There is a different kind of method that I like to call "Injectable".  These methods allow you to addon additional parameters, and those parameters will be injected by the DI container.  These are special :snowflake: methods.

These special methods are designed such that you can avoid directly using `IServiceProvider` on the `IApplicationBuilder` or your middleware.  To avoid the pitfalls with just using Service Location.

* The `Startup.Configure` method is one such method.<br/>
This allows you to inject services that you need to configure your pipeline.

```csharp
public void Configure(IApplicationBuilder app, IServiceA serviceA) { ... }
```

* Middleware is another.<br />
This allows you to include scoped services.

```csharp
public void Configure(IApplicationBuilder app, IServiceA serviceA) {
    app.Run<IServiceA>((httpContext, serviceA) => { ... });
}
// OR
public class MyMiddleware {
    public RequestServicesContainerMiddleware(RequestDelegate next, IServiceA service) {}
    public async Task Invoke(HttpContext httpContext, IScopedService scopedService) {
        ...
    }
}
```

### Using your own DI Container
Yes you can! In fact it super easy to use your own container or implement your own `ISeviceProvider`.  The easiest way is to modify your `Startup.ConfigureServices` method to return `IServiceProvider`.  This is the Service Provider that is attached with your application builder, and used from that point forward.  An example for how Autofac does it, is below.  Also in this example, you can now use `Lazy<>` or `Func<>` in your service dependencies.

```csharp
public IServiceProvider ConfigureServices(IServiceCollection services) {
    ...
    var builder = new ContainerBuilder();
    builder.Populate(services);
    var container = builder.Build();
    return container.Resolve<IServiceProvider>();
}
```

### Ripping back the curtain
Dependency Injection in DNX is around using [Service Location](http://en.wikipedia.org/wiki/Service_locator_pattern).  While service location is useful, it is generally frowned upon using it when you don't need to, instead you use something like Constructor Injection.  When you use service location by itself, instead of using constructor injection, it tends to lead toward code that is harder to maintain, understand and unit test properly.  With `Constructor Injection` and `Injectables` you should never have to even know that Service Location exists.  If you are a library author understanding how things work, is also very helpful.

The AspNet team is using the existing [IServiceProvider](https://msdn.microsoft.com/en-us/library/system.iserviceprovider(v=vs.110).aspx) Interface, which offers the method `GetService(Type type)`.  Ask it for your type, and it will return an instance of that service (:warning: or null if no type is registered).

The first place you can encounter `IServiceProvider` is on `IApplicationBuilder` of your `Startup` class.

```csharp
public void Configure(IApplicationBuilder app) {
    app.ApplicationServices.GetService(...);
}
```

`IServiceProvider` is also available on `HttpContext.ApplicationServices` and also for request based services as `HttpContext.RequestServices`.  In the case of request services, this can be services that are different per-request (such as database sessions).



### FIN
I hope you enjoyed your brief look at Dependency Injection with DNX, and hope you gained some valuable knowledge!

If you notice anything is wrong :interrobang:, not explained well enough, or you have additional questions, please feel free to leave a comment or reach out on [Jabbr](https://jabbr.net/) ( #AspNetvNext ) or [Twitter](https://twitter.com/david_blacklite).
