# aspnetcore5-dependency_injection
ASP.NET Core 5 - Dependency Injection


## Dependency Injection (DI)

### Multiple implementation for same Interface

In SimpleInjector, this was very easy:

```cs
container.Collection.Register<IMyService>(new[] { typeof(ServiceA) });
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
