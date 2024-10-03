# Mediator Pattern

## Understanding the Mediator Pattern

The Mediator Pattern is a behavioral design pattern used to simplify communication between different components or objects in a system. It helps reduce the direct dependencies between these components by centralizing their communication through a "mediator" object. Let’s break it down in simple terms.

### Problem Without Mediator

Imagine you have several components (like a user form, notification service, and data logger) that need to communicate with each other. If every component directly calls each other, it creates a spaghetti code situation. With each new component, the dependencies increase exponentially, making the code hard to manage and error-prone. 

### Mediator Pattern to the Rescue

The Mediator Pattern introduces a single mediator object that controls the communication between all components. Now, instead of talking to each other directly, the components communicate through the mediator. This pattern is especially helpful when you have multiple components that are highly coupled and constantly interact with each other. The mediator decouples these interactions, making the code more maintainable and organized. The mediator handles the logic to decide who should react to which event, making the communication flow centralized.

## Understanding Mediator in the Context of CQRS

In a typical CQRS-based architecture, you need a way to send commands and queries to the appropriate handlers without the components directly depending on each other. This is where the Mediator Pattern comes into play.

#### Mediator Pattern helps manage the request-response flow: 
- It acts as a dispatcher that routes commands and queries to their respective handlers.
- It abstracts away the details of which handler is responsible for processing a given request, so you don’t have to worry about wiring up dependencies manually.

### How Mediator Works in CQRS

- **Command or Query is Sent:** A client (like a controller or service) issues a command or query to perform a certain operation.
- **Mediator Receives the Request:** Instead of directly invoking a handler, the command/query is sent to the Mediator.
- **Mediator Finds the Appropriate Handler:** The mediator figures out which handler is responsible for the given request.
- **Handler Processes the Request:** The appropriate handler processes the command or query and performs the necessary logic (e.g., updating the database, returning data).

### Example using MediatR in .NET

In .NET, this can be efficiently implemented using the MediatR library, which is widely used for the Mediator Pattern in CQRS.

Install MediatR and Dependencies

```bash
dotnet add package MediatR.Extensions.Microsoft.DependencyInjection
```

### Example 

Suppose we have a system to create a new user and get a list of users. Here’s how the code would look using the MediatR library. 

**1. Create Command and Query Classes (CQRS Pattern)**

```csharp
// Command to create a new user
public class CreateUserCommand : IRequest<Guid>
{
    public string Username { get; set; }
    public string Email { get; set; }
}

// Query to get a list of users
public class GetUsersQuery : IRequest<List<UserDto>>
{
}
```

**2. Create Handlers for Commands and Queries**
```csharp
// Handler for the CreateUserCommand
public class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, Guid>
{
    // implementations
}

// Handler for the GetUsersQuery
public class GetUsersQueryHandler : IRequestHandler<GetUsersQuery, List<UserDto>>
{
    // implementations
}
```

**3. Register Mediator and Handlers in `DependencyInjection.cs`**
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddMediatR(typeof(DependencyInjection)); // Register MediatR

    // Register Repositories and Handlers
    services.AddScoped<IUserRepository, UserRepository>();
}
```

**4. Use Mediator in the Controller**
```csharp
[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    private readonly IMediator _mediator;

    public UsersController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<IActionResult> CreateUser(CreateUserCommand command)
    {
        var userId = await _mediator.Send(command);
        return Ok(userId);
    }

    [HttpGet]
    public async Task<IActionResult> GetUsers()
    {
        var users = await _mediator.Send(new GetUsersQuery());
        return Ok(users);
    }
}
```

### How It All Fits Together
- When `CreateUserCommand` is sent using `_mediator.Send(command)`, the Mediator finds the CreateUserCommandHandler and calls its Handle method.
- Similarly, when `GetUsersQuery` is sent, the Mediator finds the `GetUsersQueryHandler` and returns the list of users.

### Why Use Mediator in CQRS?
- Decoupling: The Controller doesn’t need to know which handler processes a command or query. 
- Centralized Dispatching: All the logic for deciding which handler to call is centralized, reducing code duplication.
- Single Responsibility Principle: Each handler has one responsibility (either handle a command or a query), keeping the code clean and maintainable.