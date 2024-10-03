# Dependency Injection Service Lifetime 

## Understanding Service Lifetimes in .NET Core
When you use Dependency Injection (DI) in .NET Core, one of the key concepts to understand is service lifetimes. This concept dictates how long an object (or service) will live once it is created by the DI container. Each of these lifetimes determines when the DI container will create a new instance of a service or reuse an existing one. There are three main lifetimes in .NET Core:

- Singleton
- Scoped
- Transient

### Singleton: "One for the Entire Application"

- **What It Means**: There will be one instance of the service for the entire lifetime of the application. Every time you request this service, the same instance will be returned.
- **Real-Life Analogy**: Imagine a single coffee pot in your café that everyone uses to get their coffee. No matter how many people request coffee, they’ll all get it from the same pot.
- **When to Use It**: Use when you need a single, shared resource that is expensive to create, such as a configuration manager, logging service, or a database connection pool.

Every time IConfigurationService is requested, the same instance of ConfigurationService will be returned.

**Caution: Since a singleton instance lives throughout the application’s lifetime, be mindful of shared state. If multiple threads access it at the same time, it can lead to data consistency issues.**

### Scoped: "One Per Request"

- **What It Means**: A new instance of the service is created once per HTTP request. So, if your application handles 100 requests, each request will get its own fresh instance of the service.
- **Real-Life Analogy**: Imagine each table in your café has its own unique coffee pot. If a customer sits at a table, they get their own coffee pot that is exclusive to them until they leave. Once they leave, that pot is discarded.
- **When to Use It**: Use it for per-request services, such as database contexts (DbContext) or services that should keep data isolated for each user session.

Each new HTTP request will get a fresh instance of OrderService.

### Transient: "One for Every Use"

- **What It Means**: A new instance of the service is created every time it’s requested. This means if a service is requested 10 times, 10 different instances will be created.
- **Real-Life Analogy**: Imagine a café where a new cup of coffee is brewed from scratch for every single customer, even if two customers place the same order.
- **When to Use It**: Use for lightweight, stateless services or for short-lived operations. Transient services are ideal for utility operations, like formatting or lightweight processing.

Each time IEmailService is requested, a brand-new instance of EmailService will be returned.

**Caution: Avoid using transient services if creating them is costly. It can lead to performance overhead.**

```csharp
public void ConfigureServices(IServiceCollection services) {
    services.AddSingleton<IPizzaMenuService, PizzaMenuService>();  // Singleton
    services.AddScoped<IOrderService, OrderService>();             // Scoped
    services.AddTransient<IDeliveryPersonService, DeliveryPersonService>();  // Transient
}
```

### Choosing the Right Lifetime
The choice between these lifetimes depends on the behavior you want for your application:

- **Singleton**: Use when you need a single, long-lived instance that is shared across the entire app (e.g., configuration service, logging, or caching).
- **Scoped**: Use when you want to isolate data or instances per request (e.g., database contexts, business logic services tied to a specific session).
- **Transient**: Use when you want a new instance for every operation (e.g., stateless services or lightweight operations).

### Example Scenario: Building a Pizza Delivery App
Imagine we are building a simple pizza delivery app. Here’s how we might use these lifetimes:

- **Singleton**: A PizzaMenuService that contains all available pizzas. Since the menu rarely changes, you want a single shared instance for the whole app.
- **Scoped**: A OrderService that handles each customer’s pizza order. You want a separate OrderService for each request, so customers don’t interfere with each other’s orders.
- **Transient**: A DeliveryPersonService that calculates delivery time and sends notifications. This service is stateless and lightweight, so creating a new instance for each use is okay.