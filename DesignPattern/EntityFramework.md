# Entity Framework 

## Understanding the Concept 

Imagine you’re building an app that needs to store information about books in a database. Writing SQL queries to insert, update, or delete data can be repetitive and complex, especially as your app grows. Entity Framework (EF) simplifies this by acting as a bridge between your C# code and the database.

Think of Entity Framework as a translator between two people who speak different languages. Your application "speaks" C#, and the database "speaks" SQL. EF converts C# commands into SQL and vice versa, so they can communicate without you writing all that database-specific code.

### Entities (C# Classes) 

These represent your database tables in code. For example, you might have a **Book class** that matches the structure of the **Books table** in your database.

```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
}
```
EF will take this `Book` class and automatically map it to a table in the database, where each property (like Title, Author) is mapped to a column.

### DbContext (The Connection)

This is like your manager who talks to both the application and the database. It’s responsible for managing the connection to the database, tracking changes, and saving data.

```csharp
public class ApplicationDbContext : DbContext
{
    public DbSet<Book> Books { get; set; } // This DbSet represents the Books table
}
```
The `DbSet<Book>` is like a collection of books that you can query and modify, and EF will handle the actual database operations behind the scenes.

### CRUD Operations with EF
With EF, you can perform common database operations (Create, Read, Update, Delete) directly using your C# objects.

```csharp
// Create 
var newBook = new Book { Title = "Clean Architecture", Author = "Robert C. Martin" };
_context.Books.Add(newBook);  // Adds a new book
_context.SaveChanges();       // Saves changes to the database

// Read
var book = _context.Books.Find(1);  // Finds a book by its ID

// Update
var book = _context.Books.Find(1);  
book.Title = "Updated Title";      
_context.SaveChanges();            // Saves changes

// Delete
var book = _context.Books.Find(1);  
_context.Books.Remove(book);        
_context.SaveChanges();             
```

**EF core automatically creates a junction table to manage the many-to-many relationship between Entities. You don’t have to define this manually (EF5+).**