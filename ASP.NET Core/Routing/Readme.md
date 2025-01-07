# [Routing](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing?view=aspnetcore-9.0)
Is the process of mapping a request URL path such as /Orders/1 to some handler that generates a response.

## From ASP.NET Core 3.0
The routing step is separate from the invocation of the endpoint. In practical terms that means we have two pieces of middleware:

- EndpointRoutingMiddleware that does the actual routing i.e. calculating which endpoint will be invoked for a given request URL path.
- EndpointMiddleware that invokes the endpoint.

```C#
public void Configure(IApplicationBuilder app)
{
    app.UseStaticFiles();

    // Add the EndpointRoutingMiddleware
    app.UseRouting();

    // All middleware between UseRouting and UseEndpoints will know which endpoint will be invoked
    // can be benerfits from it to do some logical checks
    // For instances:
    app.UseCors();
    app.UseAuthentication();
    app.UseAuthorization();

    // Execute the endpoint selected by the routing middleware
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapDefaultControllerRoute();
    });
}
```
From "UseRouting" to "UseEndpoints" is called Routing zone, some highlighted points:
- The endpoint is always null before UseRouting is called.
- If a match is found, the endpoint is non-null between UseRouting and UseEndpoints.
- The UseEndpoints middleware is terminal when a match is found.
- The middleware after UseEndpoints execute only when no match is found.

## Endpoint routing
You can attach metadata to endpoints so intermediate middleware (e.g. Authorization, CORS) can know what will be eventually executed
For example: MapHealthChecks

```C#
app.UseAuthentication();
app.UseAuthorization();

app.MapHealthChecks("/healthz").RequireAuthorization();
```

### How to create a endpoint routing aware:
 - Write an extension method on IEndpointRouteBuilder.
 - Create a nested middleware pipeline using CreateApplicationBuilder.
 - Attach the middleware to the new pipeline.
 - Build the middleware pipeline into a RequestDelegate.
 - Call Map and provide the new middleware pipeline.
 - Return the builder object provided by Map from the extension method.

### [Fully example is in Github](https://github.com/GiangHM/PracticalASPNet/tree/main/PracticalAPI/RouterwareSamples)
