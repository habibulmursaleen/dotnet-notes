# Dependency Injection (DI) 

## Scenario: A Real-World Example — Pizza Delivery Service

Let’s take a scenario that might be easier to visualize: a Pizza Delivery Service.

### Without Dependency Injection
Imagine you own a pizza delivery service with just one delivery guy named John. You call him every time you get an order. If John is on vacation, the service cannot operate until he’s back.

Similarly, in a program, this is when one class is directly dependent on another specific class, making it hard to change or replace.

```csharp
public class PizzaService {
    private readonly JohnTheDeliveryGuy _deliveryGuy;

    public PizzaService() {
        _deliveryGuy = new JohnTheDeliveryGuy();  // Hard-coded dependency
    }

    public void DeliverOrder(string address) {
        _deliveryGuy.Deliver(address);
    }
}
```
Here, `PizzaService` can only use `JohnTheDeliveryGuy` and no one else.

### Using Dependency Injection

Now imagine you decide to hire delivery guys on a contract basis, so you can call anyone who’s available. This time, you won’t depend on John alone. You’ll have a flexible service where you just call a delivery person (whoever is available).

Similarly, you can use Dependency Injection to decouple `PizzaService` from `JohnTheDeliveryGuy` by using an interface `IDeliveryPerson`:

```csharp
public interface IDeliveryPerson {
    void Deliver(string address);
}

public class PizzaService {
    private readonly IDeliveryPerson _deliveryPerson;

    public PizzaService(IDeliveryPerson deliveryPerson) {
        _deliveryPerson = deliveryPerson;  // Injected dependency
    }

    public void DeliverOrder(string address) {
        _deliveryPerson.Deliver(address);
    }
}
```

## Understanding Dependency Injection (DI) Concept 

Dependency Injection (DI) is a design pattern that allows a class to receive its dependencies from an external source, rather than creating them directly. In other words, instead of a class creating the objects it needs to work, those objects (or dependencies) are "injected" into the class, usually via a constructor or method.

**Key Terms**
- **Dependency**: An external object or service that a class relies on to perform its job. For example, a class that retrieves data from a database might depend on a Repository or DbContext.
- **Injection**: Providing a class with the dependencies it needs instead of creating them within the class.

### Why Use Dependency Injection?

**Imagine a scenario without DI:**

```csharp
public class OrderService {
    private readonly OrderRepository _orderRepository;

    public OrderService() {
        _orderRepository = new OrderRepository();
    }
}
```
In the above example, `OrderService` is tightly coupled with `OrderRepository`. If `OrderRepository` changes, you might need to modify `OrderService` as well. 


**Now, with DI:**

```csharp
public class OrderService {
    private readonly IOrderRepository _orderRepository;

    public OrderService(IOrderRepository orderRepository) {
        _orderRepository = orderRepository;
    }
}
```
In this version, `OrderService` depends on an interface `IOrderRepository` and expects it to be provided when `OrderService` is created.


## Why is This Important in Larger Applications?

As your application grows, you’ll have dozens or even hundreds of classes interacting with each other. Imagine if all of them were tightly coupled to specific implementations. Making even a small change would require a lot of rewriting.

## Implementing DI in .NET Core with Clean Architecture and Domain-Driven Design (DDD)

.NET Core has built-in support for DI using the `Microsoft.Extensions.DependencyInjection` namespace. You define dependencies in a special class called the `DependencyInjections.cs`.

Dependency Injection fits in by allowing each layer to depend only on abstractions, not on the concrete implementations of lower layers. For example, a business rule in the Domain Layer should never directly access a database. Instead, it uses an interface like IOrderRepository, and the Infrastructure Layer provides the concrete implementation. This decoupling is made possible using DI.

### Step 1: Define Interfaces and Implementations

```csharp
// we have an interface for a repository
// Domain Layer
public interface IOrderRepository {
    void AddOrder(Order order);
}

// And its implementation
// Infrastructure Layer
public class OrderRepository : IOrderRepository {
    public void AddOrder(Order order) {
        // Logic to add order to the database
    }
}
```

### Step 2: Register Services in the `DependencyInjections.cs`

```csharp
public void ConfigureServices(IServiceCollection services) {
    // Register the repository
    services.AddScoped<IOrderRepository, OrderRepository>();

    // Register the service that uses the repository
    services.AddScoped<OrderService>();
}
```

### Step 3: Use the Registered Dependencies
In the controller or service, you can now use these registered services:
```csharp
public class OrderController : ControllerBase {
    private readonly OrderService _orderService;

    public OrderController(OrderService orderService) {
        _orderService = orderService;
    }

    [HttpPost]
    public IActionResult CreateOrder(Order order) {
        _orderService.AddOrder(order);
        return Ok();
    }
}
```

