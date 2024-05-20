a. What is middleware
	• Middleware is software added to pipeline to process request and response.
	• When request come, it go through one by one middleware (you can think that all middleware do common process for each request, response)
	• To use middleware in IApplicationBuilder, use:
		○ Run: param: context
		○ Use: param: context, next -> to invoke next middleware
		○ Map: to branch middleware
	• We have some built-in middleware, we can use them in configure method.
	• Of course, we can also create custom middleware.

b. Order of middleware
c. How to create a custom middleware
	• step 1: Encapsulate custom middleware in class
		○ Convention: {Name}Middleware
		○ Include:
			§ a constructor with parameter type is "ContextDelegate".
			§ A public method named Invoke or InvokeAsync. This method must:
				□ Return a Task.
				□ Accept a first parameter of type HttpContext.
				□ Additional parameters for the constructor and Invoke/InvokeAsync are populated by dependency injection (DI).
		○ For example:
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
	• step 2: create a middleware extension method through IApplicationBuilder:
		○ Name convention: Use{MiddlewareName}
using Microsoft.AspNetCore.Builder; 
namespace Culture 
{ 
public static class RequestCultureMiddlewareExtensions 
{ 
public static IApplicationBuilder UseRequestCulture( 
this IApplicationBuilder builder) 
{ 
return builder.UseMiddleware<RequestCultureMiddleware>(); 
} 
} 
}
	• step 3: Call middleware in Configure method
public class Startup 
{ 
public void Configure(IApplicationBuilder app) 
{ 
app.UseRequestCulture(); 
app.Run(async (context) => 
{ 
await context.Response.WriteAsync( 
$"Hello {CultureInfo.CurrentCulture.DisplayName}"); 
}); 
} 
}

References: 
https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write?view=aspnetcore-5.0#middleware-class
https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-5.0#middleware-order
