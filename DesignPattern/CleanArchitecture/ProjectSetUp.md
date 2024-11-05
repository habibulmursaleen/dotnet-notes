# Porject Setup and Run Application

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) or later
- [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) (or any database of your choice)
- [Visual Studio](https://visualstudio.microsoft.com/) or [Visual Studio Code](https://code.visualstudio.com/) (Recommended for development)
- Docker

### Architecture Overview

This project is structured based on Clean Architecture, with separation into the following main layers:

- **Core** - Contains the Domain layer, including entities, aggregates, value objects, and domain services.
- **Application** - Contains use cases, DTOs, and interfaces.
- **Infrastructure** - Contains the implementations of repositories, data access, and integrations.
- **WebApi** - The entry point for the API, handling requests and responses.

### Setup

####  Step 1: Clone the Repository

```bash
git clone https://github.com/user/exampleRepo
cd NorskApi
```

####  Step 2: Restore NuGet Packages
Run the following command to restore the necessary NuGet packages:

```bash
dotnet restore
```

####  Step 3: Set Up Docker and Database
Ensure your SQL Server is running. Update the connection string in appsettings.Development.json located in the Api project:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=norskapi;User Id=your_userId;Password=your_password;Pooling=true;Min Pool Size=10;Max Pool Size=200;Connection Lifetime=180;Connection Timeout=30;Encrypt=false;"
  }
}
```
Go to the root of the folder and run - (if you already setup the Docker in your pc, you can skip this)

```bash
docker network create mssqlnetwork 
docker compose up
```

####  Step 3: Apply Migrations
Navigate to the Infrastructure layer where the DbContext is located and run the following commands to apply migrations:
```bash
dotnet ef migrations add Init -p src/NorskApi.Infrastructure -s src/NorskApi.Api; 
```

To add a new migration, run the following command:

```bash
dotnet ef migrations add MigrationName -p src/NorskApi.Infrastructure -s src/NorskApi.Api; 
```

Use the following command to apply the existing migrations and update the database:

```bash
dotnet ef database update -p src/NorskApi.Infrastructure -s src/NorskApi.Api;
```
To remove the last migration, use:

```bash
dotnet ef migrations remove -p src/NorskApi.Infrastructure -s src/NorskApi.Api;
```

####  Step 4: Build and Run the Application
Navigate to the WebApi project directory and start the API:

```bash
dotnet build
dotnet watch run --project src/NorskApi.Api   
```
#### Test

```
dotnet test
```
