# [Common](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-9.0)
- First, DI is to archive Inversion of Control, a solid principle.
- We have 3 types of DI: constructor injection, method injection, property injection
- The built-in DI of the .net core is based on constructor injection.
 ## How to register services
  - Register a service in the method 'ConfigureServices' for Asp.Net core 2,3. From version 6, we config service in the program file because of the minimal hosting model
  - Use 'services.Configure' method to config/bind  our configuration to the option
  - [Also can create an extension method of IServicesCollection to compute a group of related service](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0#register-groups-of-services-with-extension-methods)
  - Note: convention : Add{OurServiceName}, for example: AddDataAcessLayer, AddBusinessServices...
 ## Service Lifetime
  - Transient: short lifetime, created when  service container request, disposed at the end of the request
  - Scoped: create per client request, AddDbContext for example.
  - Singleton: create at the first time that they're requested; disposed of an object only when app shutdown, should consider if we use it, since memory performance
 ## [Design service for dependency injection](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines)
  - When designing services for dependency injection:
    + Avoid stateful, static classes and members. Avoid creating a global state by designing apps to use singleton services instead.
    + Avoid direct instantiation of dependent classes within services. Direct instantiation couples the code to a particular implementation.
    + Make services small, well-factored, and easily tested.
    + Dispose of services: Container takes this responsibility
   
# [From ASP.NET Core 8](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0)
 ## [Keyed Service](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0#keyed-services)
  - When using keyed services:
    + Have an interface with multiple implementations
    + And need to use one of those implementations in *different places in your application*   
  - Add services by using: AddKeyedSingleton (or AddKeyedScoped or AddKeyedTransient)
    ```C#
    builder.Services.AddKeyedSingleton<ICache, BigCache>("big");
    builder.Services.AddKeyedSingleton<ICache, SmallCache>("small");
    ```
  - Access a registered service by specifying the key with the [FromKeyedServices]
    ```C#
    [HttpGet("big-cache")]
    public ActionResult<object> GetOk([FromKeyedServices("big")] ICache cache)
    {
        return cache.Get("data-mvc");
    }
    ```

  ### [Keyed Service Example in Github repo](https://github.com/GiangHM/PracticalASPNet/tree/main/PracticalAPI/DIKeyedServices)
    

