# ASP.NET Core 5 - Dependency Injection

## Documentation

<https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-5.0>

## Lifetimes

<https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#service-lifetimes>

- Transient
    - new on every request (injection)
    - most common
- Singleton
    - 1 instance only
    - only if you have a good reason
- Scoped
    - 1x per scope ("one instance per web request")
    - hardly used
    - possible example: DBContext

## Service Registration Methods

The framework provides service registration extension methods that are useful in specific scenarios:

- Automatic object disposal
- Multiple implementation for same Interface
- Pass Args

<https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#service-registration-methods>

## Registration Factory

```cs
// IOtherService get intatiated not before IService is requested
services.AddTransient<IService>(sp => new ServiceImpl(sp.GetRequiredService<IOtherService>));
```

## ActivatorUtilities

<https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.activatorutilities?view=dotnet-plat-ext-5.0>

## Decorators in .NET Core with Dependency Injection

<https://greatrexpectations.com/2018/10/25/decorators-in-net-core-with-dependency-injection>

## Generic Type registration

```cs
IServiceCollection services = new ServiceCollection();
 
services.AddTransient<MyClassWithValue>();
services.AddTransient(typeof(IMyGeneric<>), typeof(MyGeneric<>));
 
var serviceProvider = services.BuildServiceProvider();
 
var service = serviceProvider.GetService<IMyGeneric<MyClassWithValue>>();
```

## Multiple implementation for same Interface

Examples: <https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#service-registration-methods>

Definition:

```cs
services.AddSingleton<IMessageWriter, ConsoleMessageWriter>(); // other lifetime allowed
services.AddSingleton<IMessageWriter, LoggingMessageWriter>(); // other lifetime allowed
```

Injection:

```cs
    public class ExampleService
    {
        public ExampleService(
            IMessageWriter messageWriter,
            IEnumerable<IMessageWriter> messageWriters)
        {
            Trace.Assert(messageWriter is LoggingMessageWriter);
            var dependencyArray = messageWriters.ToArray();
            Trace.Assert(dependencyArray[0] is ConsoleMessageWriter);
            Trace.Assert(dependencyArray[1] is LoggingMessageWriter);
        }
    }
```

## Multiple implementation for same Interface (specific version)

In SimpleInjector, this was very easy:

```cs
container.Collection.Register<IMyService>(new[] { typeof(ServiceA), typeof(ServiceB), typeof(ServiceC) });
```

Usage:

```cs
public NotificationProcessor(IMyService[] producers)
    {...}
```

In the built-in Dependency Injection of ASP.NET Core 3, we need a little workaround for this:

```cs
public delegate IMyService MyServiceResolver(string key);
```

Then in your Startup.cs, setup the multiple concrete registrations and a manual mapping of those types:

```cs
services.AddTransient<ServiceA>();
services.AddTransient<ServiceB>();
services.AddTransient<ServiceC>();

services.AddTransient<MyServiceResolver>(serviceProvider => key =>
{
    switch (key)
    {
        case "A":
            return serviceProvider.GetService<ServiceA>();
        case "B":
            return serviceProvider.GetService<ServiceB>();
        case "C":
            return serviceProvider.GetService<ServiceC>();
        default:
            throw new KeyNotFoundException(); // or maybe return null, up to you
    }
});
```

And use it from any class registered with DI:

```cs
public class MyConsumer
{
    private readonly IMyService _aService;

    public Consumer(MyServiceResolver serviceAccessor)
    {
        _aService = serviceAccessor("A");
    }

    public void UseServiceA()
    {
        _aService.DoTheThing();
    }
}
```

## Static class injection

Source: <https://www.strathweb.com/2016/12/accessing-httpcontext-outside-of-framework-components-in-asp-net-core/>

(indirectly) use DI in a static class:

```cs
public static class MyStaticLib
{
    private static IHttpContextAccessor _httpContextAccessor;
    public static void Configure(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }
    public static HttpContext HttpContext => _httpContextAccessor.HttpContext;
    // ...
}
```

`DI helper` method:

```cs
public static class AppConfig
{
    public static void ConfigureMyStaticLib(IApplicationBuilder app)
    {
        var httpContextAccessor = app.ApplicationServices.GetRequiredService<IHttpContextAccessor>();
        Configure(httpContextAccessor);
    }
}
```

Call `DI helper` in Startup.cs:

```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...
    ConfigureMyStaticLib(app);
    //or app.ConfigureMyStaticLib(); if you set it up as an extension
}
```

## Parameter injection

```cs
public IActionResult Index([FromServices] IService svc)
{
}
```

## Information

- DI in .NET: <https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-usage>
- DI Guidelines: <https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines>
- Configuration: https://github.com/boeschenstein/aspnetcore3-configuration
