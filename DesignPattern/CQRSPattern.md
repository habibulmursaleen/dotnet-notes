# CQRS Pattern

## Understanding the CQRS Concept
CQRS (Command Query Responsibility Segregation) is a design pattern that separates the read and write operations of an application into distinct models. This separation isolates the responsibilities of querying and modifying data.

Imagine you have a single class or service that both reads data from a database and modifies it. Over time, this class can become bloated, making it harder to maintain and extend. CQRS helps by dividing this single responsibility into two separate paths:

- **Command**: Changes or modifies (Adds, updates, or deletes) the system’s state.
- **Query**: Reads data without making any changes and returns data to the user. 

### How CQRS Works

To be clear, CQRS only handles the requests, not actually encounter with the database. For example - it will handle command requests like “Create a new product.” or "Update the product" or "Delete the product" otherwise it will handle Query requests like "Get the list of all products". It does not perform anything like create/update/delete in the database.  

- **Command Side**: You send a command (e.g., CreateOrderCommand). The system processes the command, which updates the database or changes the state of the system.
- **Query Side**: You send a query (e.g., GetOrderDetailsQuery). The system retrieves the data and returns it without making any changes.

### Code Example of CQRS in .NET Core 

Suppose you have a simple system that manages Products. Here’s how you would structure the code using CQRS principles:

```csharp
// Command to Create a New Product
public class CreateProductCommand : IRequest<Guid>
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Command Handler
public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, Guid>
{
     
}

// Query to Get All Products
public class GetProductsQuery : IRequest<List<ProductDto>>
{
}

// Query Handler
public class GetProductsQueryHandler : IRequestHandler<GetProductsQuery, List<ProductDto>>
{
     
}
```

#### Using the Commands and Queries

Now, you would use a Mediator (e.g., with MediatR library) to send these commands and queries from your controller:

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    private readonly IMediator _mediator;

    public ProductsController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<IActionResult> CreateProduct([FromBody] CreateProductCommand command)
    {
        var productId = await _mediator.Send(command);
        return Ok(productId);
    }

    [HttpGet]
    public async Task<IActionResult> GetProducts()
    {
        var products = await _mediator.Send(new GetProductsQuery());
        return Ok(products);
    }
}
```