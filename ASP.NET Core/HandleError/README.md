# [Error Handling](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-9.0)

There are many way to handle error when working with ASP.Net Core: Developer exception page, Exception handler page, Exception handler lambda...
But Developer usually prefer a global approach for handling error.

## Before ASP.Net 8.0:
Developer ofter use a CustomMiddleware for this purpose. I have an example of this approach - a [CustomExceptionHandlerMiddleware.cs](https://github.com/GiangHM/PracticalASPNet/blob/main/PracticalAPI/CustomMiddleware/CustomExceptionHandlerMiddleware.cs)

## Starting from version 8.0
They can use a new way is: [IExceptionHandler](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-9.0#iexceptionhandler)

The built-in exception handler middleware uses IExceptionHandler implementations to handle exceptions.

This interface has only one TryHandleAsync method.
+ If Exception can handle => return true;
+ If Not => return false
+ Can implement a custom logic in the method

An implement of IExceptionHandler

```C#
public class GlobalExceptionHandler : IExceptionHandler
    {
        private readonly ILogger<GlobalExceptionHandler> logger;

        public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
        {
            this.logger = logger;
        }

        public async ValueTask<bool> TryHandleAsync(
            HttpContext httpContext,
            Exception exception,
            CancellationToken cancellationToken)
        {
            this.logger.LogError(exception, "Exception occurred: {Message}", exception.Message);

            var problemDetails = new ProblemDetails
            {
                Status = StatusCodes.Status500InternalServerError,
                Title = "Server error -- apply IExceptionHandler for Global"
            };

            httpContext.Response.StatusCode = problemDetails.Status.Value;

            await httpContext.Response
                .WriteAsJsonAsync(problemDetails, cancellationToken);

            return true;
        }
    }
```

Use the handler in Program file

```C#
// User Exception Handler
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
```

### Chain exception handlers
We can add multiple IExceptionHandler implementations, and they're called in the order they are registered. A possible use case for this is using exceptions for flow control.

New exception handler

```C#
public class BadRequestExceptionHandler : IExceptionHandler
    {
        private readonly ILogger<BadRequestExceptionHandler> _logger;

        public BadRequestExceptionHandler(ILogger<BadRequestExceptionHandler> logger)
        {
            _logger = logger;
        }

        public async ValueTask<bool> TryHandleAsync(
            HttpContext httpContext,
            Exception exception,
            CancellationToken cancellationToken)
        {
            if (exception is not BadRequestException badRequestException)
            {
                return false;
            }

            _logger.LogError(
                badRequestException,
                "Exception occurred: {Message}",
                badRequestException.Message);

            var problemDetails = new ProblemDetails
            {
                Status = StatusCodes.Status400BadRequest,
                Title = "Bad Request",
                Detail = badRequestException.Message
            };

            httpContext.Response.StatusCode = problemDetails.Status.Value;

            await httpContext.Response
                .WriteAsJsonAsync(problemDetails, cancellationToken);

            return true;
        }
    }
```

Register the handlers in Program file

```C#
// User Exception Handler
builder.Services.AddExceptionHandler<BadRequestExceptionHandler>();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
```

In conclusion, ASP.NET Core 8 marks a pivotal advancement in error handling, introducing the IExceptionHandler interface to replace middleware-based approaches with a more cohesive and flexible error management system. This new paradigm not only simplifies the developer's task in crafting custom exception handling strategies but also enhances the application's ability to respond dynamically to a wide range of errors.
