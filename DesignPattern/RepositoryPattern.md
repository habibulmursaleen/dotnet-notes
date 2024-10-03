# Repository Pattern

## Understanding of Repository pattern 

The Repository Pattern is a design pattern that acts like a middleman between your application’s business logic and the data source (like a database). It helps organize how you access data by separating the data access logic from the business logic.

**Imagine this scenario:**

You own a library, and you hire a librarian. Now, whenever someone wants to borrow a book, they don't go searching through all the shelves by themselves; instead, they just ask the librarian. The librarian knows exactly where to look and will give them the right book.

If you rearrange the library or move to a new building, the way people interact with the librarian doesn’t change. They still ask the librarian for books, and the librarian handles the details. The people borrowing the books don’t need to know anything about how the library is organized or where the books are stored.

In Domain-Driven Design (DDD) and Clean Architecture, you want to keep your core business logic (the domain) separate from how data is stored and retrieved. The Repository Pattern helps with that by making sure the core logic doesn't need to know the details of how data is fetched or saved.

**The Repository Pattern fits into the Infrastructure Layer.** It helps abstract away the details of how data is stored or retrieved. The rest of the system (like the Application and Domain layers) can work with repositories without knowing if the data is coming from a database, an API, or even an in-memory cache.

### Here's the flow in simpler terms

- **Application Layer:** When your application needs data, it doesn't ask the database directly. Instead, it asks the repository for that data.

**Example**: Your `BookService` might ask the repository for a book by calling `GetBookById()`. It doesn’t know (or care) if the data is coming from a SQL database, a file, or an API.

```csharp
public interface IBookRepository
{
    Book GetById(int id);
    void Add(Book book);
    void Delete(int id);
}
```

```csharp
public class BookService
{
    private readonly IBookRepository _bookRepository;

    public BookService(IBookRepository bookRepository)
    {
        _bookRepository = bookRepository;
    }

    public void AddBook(Book book)
    {
        _bookRepository.Add(book);
    }

    public Book GetBook(int id)
    {
        return _bookRepository.GetById(id);
    }
}
```

- **Infrastructure Layer:** The repository takes that request, interacts with the database (or another data source), and gets the data. It then returns the data to the application layer.

**Example**: The `BookRepository` might use Entity Framework to look up a book in a `SQL Server database`, but the application layer only sees the result—it doesn't know how it was retrieved.

```csharp
public class BookRepository : IBookRepository
{
    private readonly ApplicationDbContext _context;

    public BookRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Book>> GetAllBooksAsync()
    {
        return await _context.Books.ToListAsync(); // Fetches all books from the database
    }

    public async Task<Book> GetBookByIdAsync(int id)
    {
        return await _context.Books.FindAsync(id); // Finds a book by its ID
    }

    public async Task AddBookAsync(Book book)
    {
        await _context.Books.AddAsync(book); // Adds a new book
        await _context.SaveChangesAsync(); // Saves changes to the database
    }

    public async Task UpdateBookAsync(Book book)
    {
        _context.Books.Update(book); // Updates the book
        await _context.SaveChangesAsync(); // Saves changes
    }

    public async Task DeleteBookAsync(int id)
    {
        var book = await _context.Books.FindAsync(id);
        if (book != null)
        {
            _context.Books.Remove(book); // Removes the book
            await _context.SaveChangesAsync(); // Saves changes
        }
    }
}
```

- **Domain Layer:** If the domain layer (core business logic) needs to validate or process the data, it does so without knowing the specifics of where the data originated. The repository hides the data access details and lets the domain layer focus on the business rules.

```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
}
```

### Registering the Repository in .NET Core

To use dependency injection (DI) in .NET Core, you need to register your repository implementation in the `DependencyInjection.cs` file. This makes the repository available wherever it’s needed in your application.

```csharp
public class DependencyInjection
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddScoped<IBookRepository, BookRepository>();  // Register repository for DI
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
        services.AddScoped<BookService>();  // Register the service
    }
}
```