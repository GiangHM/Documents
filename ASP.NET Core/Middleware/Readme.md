# What is middleware
- Middleware is software added to pipeline to process request and response.
- When request come, it go through one by one middleware (you can think that all middleware do common process for each request, response)
- To use middleware in IApplicationBuilder, use:
  + Run: param: context
  + Use: param: context, next -> to invoke next middleware
  + Map: to branch middleware
    * We have some built-in middleware, we can use them in configure method.
    * Of course, we can also create custom middleware.
# [Order of middleware](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-8.0)
![image](https://github.com/GiangHM/Documents/assets/36400582/c559a1be-85eb-4740-a869-9d3a40648383)

- If you don't call Routing Middleware explicitly, it's called at the beginning
- ## ASP .Net Core 8 sample
```C#
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using WebMiddleware.Data;

var builder = WebApplication.CreateBuilder(args);
// Add services, custom services
builder.Services.AddRazorPages();
builder.Services.AddControllersWithViews();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseMigrationsEndPoint();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
// app.UseCookiePolicy();

app.UseRouting();
// app.UseRateLimiter();
// app.UseRequestLocalization();
// app.UseCors();

app.UseAuthentication();
app.UseAuthorization();
// app.UseSession();
// app.UseResponseCompression();
// app.UseResponseCaching();

app.MapRazorPages();
app.MapDefaultControllerRoute();

app.Run();
```
- Midleware order notes:
  + UseCors, UseAuthentication, and UseAuthorization must appear in the order shown.
  + UseCors currently must appear before UseResponseCaching.
  + UseRequestLocalization must appear before any middleware that might check the request culture, for example, app.UseStaticFiles().
  + UseRateLimiter must be called after UseRouting when rate limiting endpoint specific APIs are used.
    * For example, if the [EnableRateLimiting] attribute is used, UseRateLimiter must be called after UseRouting.
    * When calling only global limiters, UseRateLimiter can be called before UseRouting.
  + Responecaching and ResponseCompression have multiple valid ordering 

# How to create a custom middleware
- Step 1: Encapsulate custom middleware in class
  + Convention: {Name}Middleware
  + Include:
    * A constructor with parameter type is "ContextDelegate".
    * A public method named Invoke or InvokeAsync. This method must:
      - Return a Task.
      - Accept a first parameter of type HttpContext.
      - Additional parameters for the constructor and Invoke/InvokeAsync are populated by dependency injection (DI), For example:
```C#
public class CustomMiddleware 
{ 
	private readonly RequestDelegate _next; 
	public CustomMiddleware(RequestDelegate next) 
	{ 
		_next = next; 
	} 
	// IMyScopedService is injected into Invoke 
	public async Task Invoke(HttpContext httpContext, IMyScopedService svc) 
	{ 
		svc.MyProperty = 1000; 
		await _next(httpContext); 
	} 
}
```
- Step 2: Create a middleware extension method through IApplicationBuilder:
  + Name convention: Use{MiddlewareName}
```C#
using Microsoft.AspNetCore.Builder; 
namespace Culture 
{ 
	public static class RequestCultureMiddlewareExtensions 
	{ 
		public static IApplicationBuilder UseRequestCulture( this IApplicationBuilder builder) 
		{ 
		return builder.UseMiddleware<RequestCultureMiddleware>(); 
		} 
	} 
}
```
- Step 3: Call middleware in Configure method
  + Net core 3.1, use in startup file
```C#
public class Startup 
{ 
	public void Configure(IApplicationBuilder app) 
	{
		// Call extension method
		app.UseRequestCulture();

		// Another way to create a inline middleware 
		app.Run(async (context) => 
	{ 
		await context.Response.WriteAsync( $"Hello {CultureInfo.CurrentCulture.DisplayName}"); }); 
	} 
}
```
   + Net core 8.0, use in program file
```C#
using Middleware.Example;
using System.Globalization;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseHttpsRedirection();

app.UseRequestCulture();

app.Run(async (context) =>
{
    await context.Response.WriteAsync(
        $"CurrentCulture.DisplayName: {CultureInfo.CurrentCulture.DisplayName}");
});

app.Run();
```
## [An example of CustomExceptionHandlerMiddleware.cs](https://github.com/GiangHM/PracticalASPNet/blob/main/PracticalAPI/CustomMiddleware/CustomExceptionHandlerMiddleware.cs)

