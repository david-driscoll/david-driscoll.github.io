---
layout: post
title: Dependency Injection with DNX
tags: aspnet aspnet5 aspnetvnext dnx coreclr di
comments: true
---

# Dependency Injection with DNX

Traditionally in AspNet applications Dependency Injection was fairly hard to pull off, and even harder to pull off properly.  In fact it usually felt a little like pulling teeth with a sledge hammer :hammer:.


## What is Dependency Injection?
If you've lived under a rock, or in the past you heard the words "XML Configuration" and promptly ran away :scream: (I don't blame you) then you might not know what Dependency Injection is about.

[Wikipedia] tells us...
> In software engineering, dependency injection is a software design pattern that implements inversion of control for software libraries. Caller delegates to an external framework the control flow of discovering and importing a service or software module. Dependency injection allows a program design to follow the dependency inversion principle where modules are loosely coupled. With dependency injection, the client part of a program which uses a module or service doesn't need to know all its details, and typically the module can be replaced by another one of similar characteristics without altering the client.

In a nut shell what it means is that instead of writing code like...

```csharp
public class MyController
{
    private readonly DbContext _context;
    public MyController()
    {
        _context = new DbContext();
    }
}
```

You write code like...
```csharp
public class MyController
{
    private readonly DbContext _context;
    public MyController(DbContext context)
    {
        _context = context;
    }
}
```

What [Inversion of Control](IoC) means is that your class no longer needs to understand how to construct it's dependencies.  Instead the Container will hand you a list of dependencies for you to use.

At first you'll be scared :scream: or perhaps even angry :angry:.  There are advantages to entering this brave new world.  

* Unit testing becomes much much easier, when you can easily stub and mock and dependencies that you are injecting.
* Code sharing is much easier, you don't have to understand where, or how you get a `DbContext` you just ask the Container for one and it will serve it up for you.
* Code is decoupled, and you no longer have to understand the inner workings of a dependency.


There are also some disadvantages to this brave new world... :disappointed:

* Code is decoupled, so you no longer understand the inner workings of a dependency.
> But you said this was a advantage?  It is both a pro and a con in some circumstances.  Generally it is a bigger advantage than disadvantage.
* Potential unseen overhead during object creation.


## What Dependency Injection looks like today
It looked something a little like the following<sup>[[source](source)]</sup>:

```csharp
public static class Bootstrapper
{
    public static void Initialise()
    {
        var container = BuildUnityContainer();

        DependencyResolver.SetResolver(new Unity.Mvc3.UnityDependencyResolver(container));

        IDependencyResolver resolver = DependencyResolver.Current;

        IDependencyResolver newResolver = new Factories.UnityDependencyResolver(container, resolver);

        DependencyResolver.SetResolver(newResolver);
    }
    ...
}
```

You have to build the container, setup the static resolver using the container, and things get worse when you want inject services into an `IHttpModule` or `IHttpHandler`.  Basically it's a lot of hoops.  It was never really fun or intuitive.  In fact it wasn't part of the original design of AspNet, so it was tacked on after, which made things very coupled.


## What Dependency Injection looks like in DNX
If you do not already know DNX is the new execution environment, standing for ".NET Execution" or ".NET Cross Platform".  It is the system that loads (or compiles) your code and hosts it.

As part of this new execution they team has already built in Dependency Injection, as a first class citizen.  Almost all of the internal code is constructed using a simple, no fancy pants, lightweight dependency injection container.  This means as soon as you start to write your code you can begin to use Dependency Injection.

### Service Location
The first time you get into your application, will be through [Service Location](ServiceLocation).

> The service locator pattern is a design pattern used in software development to encapsulate the processes involved in obtaining a service with a strong abstraction layer. This pattern uses a central registry known as the "service locator", which on request returns the information necessary to perform a certain task.

The AspNet team is using the existing [IServiceProvider](ServiceProvider) Interface, which offers the method `GetService(Type type)`.  Ask it for your type, and it will return an instance of that service.

The first place you can encounter `IServiceProvider` is on `IApplicationBuilder` of your `Startup` class.

```csharp
public void Configure(IApplicationBuilder app)
{
    app.ApplicationServices.GetService(...);
}
```

`IServiceProvider` is also available on `HttpContext.ApplicationServices` and also for request based services as `HttpContext.RequestServices`.  In the case of request services, this can be services that are different per-request (such as database sessions).

### Configuring Services
As part of your application you are going to have to register your services.  This can be achieved by adding items to the `IServiceCollection`.

In the new world, the team is calling the new model `Pay-for-play`.  That means that by default no services.  To start using a new library, you have to specifically include those services.  When consuming library services, the convention is to use an extension method on the `IServicesCollection` interface.

To use mvc you have to add the Mvc services.  This applies to all sorts of other libraries as well, such as Entity Framework, etc.
```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddMvc();
    ...
}
```

To add your own service to the services collection you can call various other methods on it.
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddTransient<IInterface, MyClass>();
}
```

This will add `IInterface` to the service collection.

#### Lifecycles
There are different lifecycles that you can give your dependencies.

* Singleton<br />
  These services will be constructed once, and from that point forward any other request for this service will return the same instance.  It is like the traditional static singleton instance pattern.
* Scoped<br />
  These services will generally be constructed once per request (there are other use cases).
* Transient<br />
  These services will be constructed every time they are requested.  Just like calling `new Service()`
* Instance<br />
  These services are registered with a specific instance, this is the singleton pattern, but the instance is constructed ahead of time.


#### Constructor Injection
By default constructor injection is used to define what services are required for a given implementation.

```csharp
public class MyService
{
    public MyService(IServiceA serviceA, IServiceB serviceB, IServiceC serviceC)
    {
        ...
    }
}
```

In the above example, the container will return instances of `IServiceA`, `IServiceB` and `IServiceC`.  There is no knowledge of the lifecycle by my service.


#### Injectables
There is a different kind of method that I like to call "Injectable".  These methods allow you to add on additional parameters, and have them injected as well.  These are special methods, and very specific methods where this is allowed.

* The `Startup.Configure` method is on such method.
```csharp
public void Configure(IApplicationBuilder app, IServiceA serviceA)
{
}
```
* Middleware is another
```csharp
app.Run<IServiceA>((httpContext, serviceA) =>
{
});
```
```csharp
public class MyMiddleware
{
    public RequestServicesContainerMiddleware(RequestDelegate next, IServiceA service) {}

    public async Task Invoke(HttpContext httpContext, IScopedService scopedService)
    {
        ...
    }
}
```

[source]: http://www.asp.net/mvc/overview/older-versions/hands-on-labs/aspnet-mvc-4-dependency-injection "Asp.Net Dependency Injection Tutorial"
[Wikipedia]: http://en.wikipedia.org/wiki/Dependency_injection "Dependency Injection"
[IoC]: http://en.wikipedia.org/wiki/Inversion_of_control "Inversion of Control"
[ServiceLocation]: http://en.wikipedia.org/wiki/Service_locator_pattern
[ServiceProvider]: https://msdn.microsoft.com/en-us/library/system.iserviceprovider(v=vs.110).aspx
