# Error Handling in .NET Core with Clean Architecture & DDD

## Exception Filter Attribute

**What It Is**: Think of it as a "safety net" for any unhandled exceptions in your Controller classes.

**Where It’s Used**: At the Controller level, mainly for catching exceptions thrown from your action methods.

**How It Works**: You add a filter attribute to your controller or globally. When an exception occurs, it catches it and allows you to log it or transform it into an appropriate response.

**Pros & Cons**:

- ✅ Good for centralized error handling in controllers.
- ❌ Only catches exceptions that are not HTTP response exceptions (e.g., HttpResponseException).

```csharp
[ApiController]
[Route("api/[controller]")]
[CustomExceptionFilter] // This attribute will catch unhandled exceptions in this controller.
public class MyController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        throw new Exception("Something went wrong!"); // This will be caught by the filter.
    }
}
```

## Middleware for Error Handling 

**What It Is**: Middleware is like a “bouncer” at the entrance of the app. It sits at the top of the request pipeline and can catch exceptions globally for the entire application.

**Where It’s Used**: At the startup level (applies to the whole application).

**How It Works**: When an exception is thrown anywhere in the application, this middleware intercepts it, logs it, and returns a user-friendly error response.

**Pros & Cons**:
- ✅ Flexible and catches exceptions from all parts of the application.
- ✅ Can handle HTTP exceptions and custom ones.
- ❌ Requires additional setup in the DependencyInjection.cs.

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseMiddleware<CustomErrorHandlingMiddleware>(); // This middleware catches all exceptions.
}
```

## Problem Details

**What It Is**: A standardized way of returning error responses defined by the RFC 7807 specification.

**Where It’s Used**: For consistent error responses in your API.

**How It Works**: When an exception occurs, you return a structured ProblemDetails object, which includes details like status, title, and description.

**Pros & Cons:**
- ✅ Helps provide a uniform error response structure.
- ✅ Makes it easy for consumers of your API to understand what went wrong.
- ❌ May require customization to fit specific needs.

```csharp
public void Configure(IApplicationBuilder app)
public IActionResult Get()
{
    var problemDetails = new ProblemDetails
    {
        Status = StatusCodes.Status500InternalServerError,
        Title = "An error occurred",
        Detail = "More detailed error information"
    };
    return new ObjectResult(problemDetails) { StatusCode = 500 };
}
```

## Error Endpoint

**What It Is**: A dedicated endpoint in your API for handling errors, often using the /error route.

**Where It’s Used**: In scenarios where you want a single place to handle all errors.

**How It Works**: Any exception in your application is redirected to this endpoint, which formats and returns the error response.

**Pros & Cons**:
- ✅ Centralized error handling.
- ❌ May require setting up a special route and redirect logic.

```csharp
app.UseExceptionHandler("/error"); // Redirect all exceptions to this route.

[Route("error")]
public IActionResult HandleError()
{
    return Problem("An error occurred");
}
```

## Custom Problem Details Factory 

**What It Is**: A way to customize how ProblemDetails objects are created and returned.

**Where It’s Used**: When you need more control over what gets included in your error responses.

**How It Works**: You extend the ProblemDetailsFactory class to include additional fields or formatting rules.

**Pros & Cons**:
- ✅ Full control over your problem details structure.
- ❌ Involves more coding and setup.

```csharp
public class CustomProblemDetailsFactory : ProblemDetailsFactory
{
    public override ProblemDetails CreateProblemDetails(HttpContext httpContext, int? statusCode = null, string title = null, string type = null, string detail = null, string instance = null)
    {
        return new ProblemDetails
        {
            Status = statusCode ?? 500,
            Title = title ?? "Custom Error",
            Detail = detail ?? "This is a custom error detail"
        };
    }
}
```

### Summary
- **Exception Filter Attribute**: Catches exceptions in Controllers.
- **Middleware**: Catches exceptions globally for the entire app.
- **Problem Details**: Standardized error response format.
- **Error Endpoint**: Centralized endpoint to handle all exceptions.
- **Custom Problem Details Factory**: Fully customizable error response generation.