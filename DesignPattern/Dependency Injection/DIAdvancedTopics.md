# Decorators

A Decorator is like an add-on feature. Imagine you have a basic pizza, but you want to add extra cheese, mushrooms, or pepperoni. Rather than changing the recipe for the basic pizza, you “decorate” it by adding extra toppings. Each topping adds some additional behavior but doesn’t alter the core pizza recipe.

In software development, a Decorator Pattern is used to add additional responsibilities to an object dynamically without modifying its structure. This means you can wrap existing functionality with new behaviors in a flexible and reusable way.

### How Decorators Fit into DI

- **In the Context of DI:** Decorators often leverage Dependency Injection to wrap existing services and extend their functionality. This means that when a service is requested from the DI container, you can inject a decorated version of it instead of the plain version. DI enables the dynamic composition of decorators, making it easy to add or remove additional behaviors.
- **Example Scenario:** Suppose you have a basic service like IOrderService, and you want to add logging without modifying its core implementation. You create a LoggingDecorator that wraps the original OrderService:

```csharp
public class LoggingOrderServiceDecorator : IOrderService {
    // The original service that this decorator wraps
    private readonly IOrderService _inner;
    
    // Logger instance to log information
    private readonly ILogger<LoggingOrderServiceDecorator> _logger;

    // Constructor to initialize the inner service and logger
    public LoggingOrderServiceDecorator(IOrderService inner, ILogger<LoggingOrderServiceDecorator> logger) {
        _inner = inner;
        _logger = logger;
    }

    // Method to place an order with logging
    public void PlaceOrder(Order order) {
        // Log before placing the order
        _logger.LogInformation("Placing order...");
        
        // Call the original PlaceOrder method
        _inner.PlaceOrder(order);
        
        // Log after the order has been placed
        _logger.LogInformation("Order placed successfully!");
    }
}
```

**DI Container Setup:**

```csharp
services.AddScoped<IOrderService, OrderService>(); // Original service
services.Decorate<IOrderService, LoggingOrderServiceDecorator>(); // Decorated with logging
```

# Middlewares

Middleware is like a security checkpoint in a building. When someone enters, they must pass through multiple gates (security, reception, etc.) before reaching their destination. Each checkpoint can either allow or block them, and even add some information (e.g., visitor badge).

In .NET Core, Middleware components are used to process incoming HTTP requests and outgoing responses. They form a pipeline, where each middleware can:

- Perform some action.
- Pass the request to the next middleware in the pipeline.
- Modify the response before sending it back to the client.

### How Middlewares Fit into DI

- **In the Context of DI:** Middleware components often rely on injected services to perform various operations. For example, a logging middleware might depend on an injected ILogger service, and an authentication middleware might rely on an injected IUserService to validate users.

- **Example Scenario:** Consider a middleware that checks if a user is logged in. This middleware might rely on a IUserSessionService to validate the user session. The DI container ensures that this service is correctly provided.

```csharp
public class AuthenticationMiddleware {
    private readonly RequestDelegate _next;
    private readonly IUserSessionService _userSessionService;

    public AuthenticationMiddleware(RequestDelegate next, IUserSessionService userSessionService) {
        _next = next;
        _userSessionService = userSessionService;
    }

    public async Task InvokeAsync(HttpContext context) {
        if (!_userSessionService.IsUserLoggedIn()) {
            context.Response.StatusCode = 401; // Unauthorized
            return;
        }
        
        await _next(context); // Pass to the next middleware
    }
}
```

DI Container Setup:

```csharp
services.AddScoped<IUserSessionService, UserSessionService>(); // Inject into middleware
```

In this example, the middleware relies on the injected service to perform authentication logic. The service lifetimes defined in DI determine how long these services will live (Scoped, Singleton, etc.).


# Interceptors

An Interceptor is like a spy in a movie. Imagine a secret agent who listens to conversations without interrupting, and occasionally manipulates a message before it reaches its destination. **The key difference between interceptors and middleware is that interceptors are focused on method calls rather than the entire request pipeline.**

In .NET Core, Interceptors are used in dependency injection to intercept method calls within a class or service. This allows you to add behaviors like logging, validation, or performance monitoring to specific methods.

### How Interceptors Fit into DI

- **In the Context of DI:**  Interceptors are often used to wrap services provided by the DI container. Using DI, you can register your service with interceptors that add additional behavior, such as logging, caching, or validation, without modifying the original service implementation.

- **Example Scenario:** Suppose you have a method in your `IOrderService` that you want to intercept for logging. You can register an interceptor that wraps the method call and logs each time the method is called.

```csharp
// Registering Interceptors via DI (using Castle DynamicProxy or a similar library):
services.AddSingleton<IInterceptor, LoggingInterceptor>(); // Register the interceptor

// Create the proxy service
services.AddSingleton(provider => {
    var generator = new ProxyGenerator();
    var orderService = provider.GetService<OrderService>();
    var loggingInterceptor = provider.GetService<LoggingInterceptor>();

    return generator.CreateInterfaceProxyWithTarget<IOrderService>(orderService, loggingInterceptor);
});

```
Here, DI is used to inject the interceptors and wrap the services, making it easy to manage which services are intercepted and which are not.

# Connecting the Dots

Decorators, Middlewares, and Interceptors — have a common goal: enhancing or modifying the behavior of a base service. But the key difference lies in how, when, and where these enhancements are applied. 

### Decorators: Enhancing Specific Service Implementations 

Decorators wrap **specific objects or services** and add additional behaviors or responsibilities to them at **runtime**. The base service itself is unaware of these decorators. This is usually done by implementing the same interface and injecting the core service into the decorator.

Imagine a basic pizza delivery service (IPizzaService) that takes orders. You want to add discount functionality for VIP users and free delivery for orders over a certain amount. These are optional behaviors that should be dynamically added.

### Middlewares: Enhancing the Entire Request Pipeline

Middlewares operate on a request-response pipeline and control the flow of HTTP requests throughout the application. They can process, filter, or halt requests before they reach a specific endpoint. They modify the entire **pipeline’s behavior** rather than focusing on specific services.

Imagine a web application where every incoming request must be authenticated. You can implement an AuthenticationMiddleware that intercepts the request, checks the authentication status, and either allows or blocks the request before it reaches the controller.

### Interceptors: Enhancing Method Calls at the Code Level

Interceptors are used to **intercept method calls on a specific service** and add extra behavior before, after, or around a method execution. They focus on granular control over **specific methods** rather than the entire service or request pipeline.

Suppose you have a TransferService with a `TransferFunds()` **method**, and you want to measure execution time for every transfer. You can use an interceptor to inject timing logic around this method. 