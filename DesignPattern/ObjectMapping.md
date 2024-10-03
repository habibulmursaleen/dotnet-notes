# Object Mapping

Imagine you have a shopping list that you write down before you go to the grocery store. It’s very simple: just a list of items like "Apples," "Bread," and "Milk."

Now, when you come back from the store, you have a receipt. It also contains "Apples," "Bread," and "Milk," but it has more details: quantities, prices, and perhaps a few items you didn't plan for. The shopping list and the receipt describe the same things, but in slightly different ways. If you want to compare them, you might want to transform one into a format that looks like the other.

This transformation from one format to another is what we call object mapping in programming.

### Why Do We Need Object Mapping?
When building software, you’ll often have data represented in different formats. Let’s say:

You have an API that sends you raw data.
You want to convert this data into a more organized format to work with in your application.
Later, you might want to convert this data again to save it in the database.

Each of these scenarios represents the same data but formatted slightly differently. Instead of manually copying each value between different formats (e.g., name, age, address), we can use object mapping to automate this process.

### Object Mapping in a Real-world Example

Imagine you have an application for managing students. You receive data from a web form that looks like this:

```json
{
    "first_name": "Alice",
    "last_name": "Johnson",
    "email": "alice@example.com"
}
```
This data structure is great for displaying in a form or showing to a user. But in your application, you might have a different way of storing student data. 

```csharp
public class Student
{
    public string FullName { get; set; }
    public string Email { get; set; }
}
```
To fit your app’s structure, you want to map the form data into a Student object:

- `first_name` and `last_name` combined into a `FullName`
- `email` remains the same.

#### Mapping Without a Tool

```csharp
// Step 1: Receive data from the form (a DTO)
var formData = new { first_name = "Alice", last_name = "Johnson", email = "alice@example.com" };

// Step 2: Map manually
var student = new Student
{
    FullName = formData.first_name + " " + formData.last_name,
    Email = formData.email
};
```

This works fine for small projects, but what if your application grows? What if you have many objects and need to convert data all the time? That’s where tools like **Mapster** come in.

## Mapster: An Object Mapping Tool

Mapster is a library that helps automate object mapping in .NET applications. Think of it like a smart assistant that automatically converts your data formats for you.

Mapster helps to keep Clean Architecture and DDD's layers separate by mapping between different models. 

- **API Layer:** Receives data as a StudentCreateDto.
- **Application Layer**: Converts it into a CreateStudentCommand.
- **Domain Layer**: Maps it to a Student entity that represents your business rules.

By using Mapster, you only define the relationships once, and then you can map between these layers automatically.


### Applying Mapster to a Simple .NET Project

Imagine we’re building a simple project that handles student data. We’ll have a form where users can submit new student information. This data will be received in one format (a `StudentCreateDto`) and then transformed into a `Student` entity using Mapster.

#### Step 1:  Install Mapster

```bash
dotnet add package Mapster
```

#### Step 2: Define the Models and DTOs

We will define three simple classes to represent the data at different stages:

- **StudentCreateDto:** This class represents the raw data received from the user.
- **CreateStudentCommand**: This class represents a command in our application layer that tells our app to create a student.
- **Student**: This is the final Student entity used in the domain or business layer.

```csharp 
// This is the DTO (Data Transfer Object) that we receive from an external source (e.g., an API or a form).
public class StudentCreateDto
{
    public string FirstName { get; set; }  // The student's first name
    public string LastName { get; set; }   // The student's last name
    public string Email { get; set; }      // The student's email address
}

// This class represents the command we use in our application layer.
// Commands are used to trigger specific actions like "Create a Student."
public class CreateStudentCommand
{
    public string FullName { get; set; }     // The student's full name, e.g., "Alice Johnson"
    public string EmailAddress { get; set; } // The student's email address
}

// This is our main Domain entity that represents a Student in our business logic.
public class Student
{
    public string FullName { get; set; } // Student's full name
    public string Email { get; set; }    // Student's email address
}
```

#### Step 3: Configure Mapster to Handle the Mapping

We want to set up Mapster to know how to transform a StudentCreateDto into a CreateStudentCommand and then into a Student. This is where we define the mapping rules.

```csharp 
using Mapster;

public class MapsterConfiguration
{
    public static void ConfigureMappings()
    {
        // Configuring the mapping from StudentCreateDto to CreateStudentCommand
        TypeAdapterConfig<StudentCreateDto, CreateStudentCommand>.NewConfig()
            .Map(dest => dest.FullName, src => src.FirstName + " " + src.LastName)  // Combine FirstName and LastName into FullName
            .Map(dest => dest.EmailAddress, src => src.Email);                     // Map Email directly

        // Configuring the mapping from CreateStudentCommand to Student entity
        TypeAdapterConfig<CreateStudentCommand, Student>.NewConfig()
            .Map(dest => dest.FullName, src => src.FullName)                        // Map FullName directly
            .Map(dest => dest.Email, src => src.EmailAddress);                      // Map EmailAddress to Email
    }
}
```
#### Use Mapster for Object Mapping in the Program

Now, let’s use Mapster in a real-world scenario. We’ll create a StudentCreateDto object, map it through different layers

Here’s the complete code for your` Program.cs`:

```csharp
using System;
using Mapster;

namespace MapsterExample
{
    class Program
    {
        static void Main(string[] args)
        {
            // Step 1: Configure the Mapster mappings
            MapsterConfiguration.ConfigureMappings();

            // Step 2: Create a StudentCreateDto object (simulating data received from a form)
            var studentDto = new StudentCreateDto
            {
                FirstName = "Alice",
                LastName = "Johnson",
                Email = "alice@example.com"
            };

            // Step 3: Map StudentCreateDto to CreateStudentCommand
            CreateStudentCommand command = studentDto.Adapt<CreateStudentCommand>();

            // Step 4: Map CreateStudentCommand to Student entity
            Student student = command.Adapt<Student>();

            // Step 5: Print the results to verify mapping
            Console.WriteLine("Mapped Student Entity:");
            Console.WriteLine($"FullName: {student.FullName}");
            Console.WriteLine($"Email: {student.Email}");

            // Output:
            // Mapped Student Entity:
            // FullName: Alice Johnson
            // Email: alice@example.com
        }
    }
}
```

#### Setting Up Mapster for Clean Architecture

```markdown
MapsterExample/
│
├── Domain/
│   └── Entities/
│       └── Student.cs
│
├── Application/
│   ├── Commands/
│   │   └── CreateStudentCommand.cs
│   └── Mappings/
│       └── MappingConfig.cs
│
├── Presentation/
│   └── Dtos/
│       └── StudentCreateDto.cs
│
├── Program.cs
└── MapsterExample.csproj
```