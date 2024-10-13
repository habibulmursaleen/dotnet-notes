# Clean Architecture

Imagine building a house. You wouldn’t want the plumbing to be tightly connected to the electrical wiring, right? You want each system (plumbing, electrical, heating) to be separate so that if one needs repair, it doesn’t affect the others. Clean Architecture is like this; it’s a way to structure your application in layers where each part has its own responsibility. Each of these systems interacts with the house (application) but remains independent, so changing the electrical wiring doesn’t affect the plumbing.

## Clean Architecture in .NET Core

In software, Clean Architecture ensures that the business logic (core of the application) remains independent from the frameworks, UI, and external services. In .NET Core with DDD, this means organizing the project into layers, where each layer is responsible for a specific job.

Here’s the general structure, starting with the outermost layers and moving inward:

#### UI Layer (Presentation)

This layer is responsible for interacting with users (e.g., through APIs, MVC controllers, or so on.Here’s the general structure, starting with the outermost layers and moving inward:

**What goes here?**

**Controllers, ViewModels, DTOs**: These help transfer data between the user and the system.

**Middlewares**: They intercept HTTP requests for things like authentication

#### Application Layer (Use Cases/Services)

This is where the application logic lives. It doesn’t care how data is presented or stored but focuses on what needs to happen.

**What goes here?**

**Interfaces**: Abstract definitions of what should happen (e.g., IOrderRepository, IUserService). The domain should not know how things are implemented.

**Services**: The logic of what needs to happen (use cases)

**CQRS (Command Query Responsibility Segregation)**: This separates how you read data (queries) from how you write data (commands).

**Error Handling**: This is also handled here, through services that manage different business outcomes.

#### Domain Layer (Core Business Rules)

This is the core of your application. It contains the business rules and logic that should not change, regardless of UI or database.

**What goes here?**

**Entities**: The core objects that represent your business (e.g., User, Order, Product). An entity is an object that is defined by its unique identity across time, rather than by its attributes. Even if two entities have the same attributes, they are considered different because of their distinct identities.

- Entities have an ID or key that uniquely identifies them, regardless of their state
- Entities have a life cycle; they are created, modified, and eventually removed.
- Entities are compared based on their identity, not their attributes.

**Value Objects**: Small, immutable objects that represent things like money or an address. A value object, on the other hand, does not have an identity. It’s defined solely by its attributes. If two value objects have the same attributes, they are considered equal.

- It is identified by its attributes rather than an ID.
- Value objects are usually immutable. Instead of modifying them, you create a new one with the new values.
- Two value objects are considered equal if all their attributes are the same.

**Aggregates**: These handle the consistency and persistence of your business objects. An aggregate is a cluster of entities and value objects that are treated as a single unit for data changes. Aggregates enforce consistency and integrity of data by ensuring that all changes to an aggregate happen through a well-defined aggregate root.

**_Entity vs Value Object:_**

**Entity**: Has a unique identity, can change over time (e.g., Customer).

**Value Object:** No identity, defined by attributes, usually immutable (e.g., Address).

**Entity vs Aggregate:**

**Entity**: A single object with a unique ID.

**Aggregate**: A group of entities and value objects managed as a single unit by an aggregate root (e.g., Order is an aggregate, and OrderItem is an entity within it).

#### Why people use value object for aggregate roots ID?

The reason people often use value objects for aggregate root IDs is because, while the aggregate root’s ID represents its identity within the system, the ID itself doesn’t have any behavior or identity of its own.

**Conceptual Simplicity:** The ID of an aggregate root is often just a unique value (like a GUID, UUID, or a specific number). The primary role of the ID is to uniquely identify the aggregate, but it doesn’t need its own identity or behavior outside of this purpose. In terms of DDD, this makes the ID a value object because:

- It has no independent lifecycle.
- It’s immutable (the ID never changes during the lifetime of the aggregate).
- Two IDs with the same value are considered the same.

```csharp
public class OrderId : ValueObject
{
    public Guid Value { get; private set; }

    public OrderId(Guid value)
    {
        Value = value;
    }
}
```

The OrderId here is a value object because it’s fully defined by its internal Guid value, and that’s all that matters—there’s no complex behavior or unique lifecycle for the OrderId itself. It doesn't change the fact that the aggregate root, Order, is an entity.

**Immutability and Safety**: Value objects are often immutable, which means once the ID is set, it cannot be changed. This is a desirable property for IDs, as an aggregate's identity should never change after it’s created. By treating the ID as a value object, we ensure immutability and thus avoid potential bugs that might arise if an ID were to be accidentally modified.

#### Infrastructure Layer (External Systems)

This layer deals with everything outside of your application (like databases, APIs, or file systems).

**What goes here?**

**Repositories**

**Implementations of Interfaces**: These are the concrete implementations of your domain interfaces (e.g., a SqlOrderRepository or MongoDbUserRepository).

**Mappers**: Convert data between your domain objects and external systems (e.g., database records).

**Decorators and Interceptors**: These add extra functionality like logging or validation around services.
).

### General Structure in .NET Core

```plaintext
- src/
  - MyProject.Api (UI Layer)
    - Controllers/
    - DTOs/
    - Middlewares/

  - MyProject.Application (Application Layer)
    - Interfaces/
    - Services/
    - CQRS/

  - MyProject.Domain (Domain Layer)
    - Entities/
    - ValueObjects/
    - DomainServices/

  - MyProject.Infrastructure (Infrastructure Layer)
    - Repositories/
    - Mappers/
    - Decorators/
```

### Data Retrieval Flow (Query Operation)

- **Step 1**: User Request --> The user sends a request to the ProjectsController (e.g., GET /api/Projects).
- **Step 2**: Controller to Service --> The ProjectsController calls the IProjectService method to get the Project data (e.g., GetAllProjectsAsync).
- **Step 3**: Service to Repository --> The ProjectService calls the IProjectRepository to fetch data (e.g., GetAllAsync).
- **Step 4**: Repository to Database --> The ProjectRepository queries the database to retrieve the Project data.
- **Step 5**: Database to Repository --> The database returns the Project data back to the ProjectRepository.
- **Step 6**: Repository to Service --> The ProjectRepository returns the data to the ProjectService.
- **Step 7**: Service to Controller --> The ProjectService sends the data back to the ProjectsController.
- **Step 8**: Controller to Client --> Finally, the ProjectsController returns the data as an HTTP response to the user/client.

#### Flow Visualization

```plaintext
Database --> ProjectRepository (Infrastructure Layer)
                                               ^
                                               |
                      IProjectRepository (Domain Layer)
                                               ^
                                               |
                              IProjectService (Application Layer)
                                               ^
                                               |
                           ProjectsController (API Layer) ----> User
```

#### Visual Representation of Response Flow

```plaintext
User <---- ProjectsController (API Layer)
              ^
              |
     IProjectService (Application Layer)
              ^
              |
     IOrderRepository (Domain Layer)
              ^
              |
   OrderRepository (Infrastructure Layer)
              ^
              |
        Database (SQL, NoSQL, etc.)
```

## Simple Project Structure

```plaintext
YourProjectName
│
├── Application
│   ├── Interfaces
│   │   ├── IOrderService.cs
│   │   └── IOrderRepository.cs
│   └── Services
│       ├── Queries
│       │    └── ProjectService.cs
│       └── Commands.cs
│           └── ProjectService.cs
├── Domain
│   ├── Entities
│   │   └── Project.cs
│   ├── Enums
│   └── ValueObjects
│
├── Infrastructure
│   ├── Repositories
│   │   └── ProjectRepository.cs
│   └── DataContext
│   └── Services
│       └── ProjectService.cs
└── API
    └── Controllers
        └── ProjectsController.cs
    └── DTOs
        ├── ProjectDto.cs
        └── CreateProjectDto.cs
```
