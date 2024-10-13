# Getting Started

## Project Create

### Step 1: Setting up the Solution Structure

First, let's set up the project with the required layers. We'll create a .NET Core solution with multiple projects corresponding to each layer.

#### 1.1 Create a New .NET Core Solution

```makrdown
dotnet new sln -o NorskApi
```

#### 1.2 Create Project Folders for Each Layer

```makrdown
dotnet new webapi -o NorskApi.Api
dotnet new classlib -o NorskApi.Contracts
dotnet new classlib -o NorskApi.Infrastructure
dotnet new classlib -o NorskApi.Application
dotnet new classlib -o NorskApi.Domain
```

#### 1.3 Add Projects to Solution

```makrdown

dotnet sln add NorskApi.Api/NorskApi.Api.csproj
dotnet sln add NorskApi.Application/NorskApi.Application.csproj
dotnet sln add NorskApi.Domain/NorskApi.Domain.csproj
dotnet sln add NorskApi.Infrastructure/NorskApi.Infrastructure.csproj

<!-- for all at once -->
 dotnet sln add $(ls **/*.csproj)\
```

#### 1.4 Set Up Project References

We need to link the layers together:

- The `Api` project should reference the `Application` project.
- The `Application` project should reference the `Domain` project.
- The `Infrastructure` project should reference both the `Application` and `Domain` projects.

```makrdown
dotnet add .\NorskApi.Api\ reference .\NorskApi.Contracts\ .\NorskApi.Application\
dotnet add .\NorskApi.Infrastructure\ reference .\NorskApi.Application\
dotnet add .\NorskApi.Application\ reference .\NorskApi.Domain\
```

#### 1.5 Install Entity Framework Core (EF Core)

```markdown
dotnet add NorskApi.Infrastructure/NorskApi.Infrastructure.csproj package Microsoft.EntityFrameworkCore
dotnet add NorskApi.Infrastructure/NorskApi.Infrastructure.csproj package Microsoft.EntityFrameworkCore.InMemory
```
