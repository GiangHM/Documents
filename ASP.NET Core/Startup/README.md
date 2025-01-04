# [ASP.Net core 3.0](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/startup?view=aspnetcore-5.0)
- Specify when the app host is built.
## ConfigureServices method:
  + Called by the host before the Configure method
  + Use to set up services for the app
  + There are some built-in extension methods (AddDbContext, AddEntityFrameworkStores..)
  + Usually used to add business services/ DAL class to dependency injection.
	  => You can write an extension method on IServiceCollection to add more services as you want.

## Configure method
  + Used to specify how the app responds to the HTTP request.
  + To configure adding  middleware component to IApplicationBuilder
  + There are some extension methods to  add middleware
  + You can write a new extension method by using IApplicationBuilder 
    => Write custom middleware

## Multiple startup
  + For different environments.
  + Suffix name matched with the current environment is prioritized,
  For example (StartupDevelopment)

# [ASP.Net Core 8](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/startup?view=aspnetcore-8.0)
  + Startup code in the program file

```C#
    var builder = WebApplication.CreateBuilder(args);
    // Add services to the container.
    builder.Services.AddRazorPages();
    builder.Services.AddControllersWithViews();
    // You can add your user-defined services here
    var app = builder.Build();
    
    // Configure the HTTP request pipeline.
    // You can add your custom middle ware here
    if (!app.Environment.IsDevelopment())
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }
    
    app.UseHttpsRedirection();
    app.UseStaticFiles();
    ...
    app.Run();
  ```
