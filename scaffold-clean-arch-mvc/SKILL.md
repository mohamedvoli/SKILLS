---
name: scaffold-clean-arch-mvc
description: 
  Scaffolds a production-ready 5-project .NET Clean Architecture solution
  using the latest installed .NET SDK, with ASP.NET Core Web API as the
  backend and ASP.NET Core MVC/Razor Views as the web presentation layer.
  Includes CQRS/MediatR, AutoMapper, FluentValidation, Entity Framework
  Core, dependency injection, API infrastructure, and Persistence inside
  Infrastructure, including foundational repository interfaces and data
  access abstractions, while creating zero business-specific features.
---------------------------------------------------------------

# 5-Project ASP.NET Core Clean Architecture

## MVC / Razor Web + Web API + CQRS + MediatR + EF Core

## Purpose

Use this skill when the user asks to initialize a new .NET application
using Clean Architecture with:

- Domain layer
- Application layer
- Infrastructure layer
- ASP.NET Core Web API
- ASP.NET Core MVC / Razor Views

The solution must be an empty but production-ready architectural
foundation.

It MUST include the technical foundations required to begin feature
development immediately.

It MUST NOT include business-specific functionality.

---

# 1. Core Architecture Principle

The architecture consists of exactly five projects:

```text
Domain
Application
Infrastructure
API
Web
```

The responsibilities are:

```text
Domain
    Business/domain concepts only
    Domain repository contracts (optional, core aggregate root rules)

Application
    Use cases
    CQRS
    MediatR
    DTOs
    Validation
    Mapping
    Application interfaces (including generic/unit of work repository interfaces)

Infrastructure
    Persistence
    EF Core
    External services
    Infrastructure implementations (DbContext, repository implementations, migrations)

API
    HTTP API
    Controllers
    API configuration
    Dependency injection composition root

Web
    MVC
    Razor Views
    ViewModels
    API clients
    User interface
```

The solution must contain zero business features.

---

# 2. Default Solution Name

If the user does not specify a solution name, use:

```text
CleanArchMvc
```

If the user specifies a name, use that name.

---

# 3. Detect .NET SDK

Before creating the solution, execute:

```powershell
dotnet --version
```

Also inspect:

```powershell
dotnet --list-sdks
```

Use the latest installed stable SDK.

Do NOT blindly assume:

```text
net8.0
net9.0
net10.0
```

Determine the appropriate target framework from the installed SDK.

---

# 4. Verify Templates

Run:

```powershell
dotnet new list
```

Verify that these templates are available:

```text
sln
classlib
webapi
mvc
```

The API MUST use:

```text
dotnet new webapi
```

The Web MUST use:

```text
dotnet new mvc
```

Do not substitute Minimal API-only architecture for the Web API project.

---

# 5. Create Solution

Create:

```powershell
dotnet new sln -n {SolutionName}
```

All projects must exist under:

```text
src/
```

---

# 6. Create Exactly Five Projects

Create exactly:

```powershell
dotnet new classlib -o src/{SolutionName}.Domain
dotnet new classlib -o src/{SolutionName}.Application
dotnet new classlib -o src/{SolutionName}.Infrastructure
dotnet new webapi -o src/{SolutionName}.API
dotnet new mvc -o src/{SolutionName}.Web
```

Do NOT create additional projects.

Do NOT create:

- Contracts project
- Shared project
- Tests project
- Persistence project
- Application.Contracts project
- API.Contracts project

unless explicitly requested by the user.

---

# 7. Add Projects to Solution

Add all five projects:

```powershell
dotnet sln {SolutionName}.sln add src/{SolutionName}.Domain/{SolutionName}.Domain.csproj
dotnet sln {SolutionName}.sln add src/{SolutionName}.Application/{SolutionName}.Application.csproj
dotnet sln {SolutionName}.sln add src/{SolutionName}.Infrastructure/{SolutionName}.Infrastructure.csproj
dotnet sln {SolutionName}.sln add src/{SolutionName}.API/{SolutionName}.API.csproj
dotnet sln {SolutionName}.sln add src/{SolutionName}.Web/{SolutionName}.Web.csproj
```

Verify that exactly five projects exist.

---

# 8. Project Dependency Graph

The dependency graph MUST be:

```text
Domain
  ↑
Application
  ↑
Infrastructure
  ↑
API
```

The Web application is a separate presentation client:

```text
API
 ↑
HTTP
 ↑
Web
```

Project references:

```text
Domain
  → none

Application
  → Domain

Infrastructure
  → Application
  → Domain

API
  → Application
  → Infrastructure

Web
  → no Domain reference
  → no Application reference
  → no Infrastructure reference
```

The Web project communicates with the API over HTTP.

Do NOT add a ProjectReference from Web to API.

Do NOT make Web a second backend composition root.

---

# 9. Domain Layer

Project:

```text
{SolutionName}.Domain
```

References:

```text
none
```

Create:

```text
Entities/
Common/
Enums/
Repositories/
```

`Repositories/` inside Domain provides a location for aggregate root contract definitions if DDD-style repository contracts are preferred at the domain level.

Remove the default class-library sample file.

Do NOT install:

- MediatR
- AutoMapper
- FluentValidation
- EF Core
- ASP.NET Core packages

The Domain project must remain framework-independent.

---

# 10. Application Layer

Project:

```text
{SolutionName}.Application
```

Reference:

```text
Domain
```

Create:

```text
Common/
    Interfaces/
        Persistence/
        Repositories/
    Models/
    Behaviors/

Features/
Mappings/
```

Application is responsible for:

- CQRS
- MediatR
- Application use cases
- DTOs
- Validation
- Mapping
- Application interfaces (including data access, generic repositories, and unit-of-work abstractions under `Common/Interfaces/Persistence/` and `Common/Interfaces/Repositories/`)
- Pipeline behaviors

No business use cases may be generated.

---

# 11. Application NuGet Packages

Install the latest stable compatible versions:

```powershell
dotnet add src/{SolutionName}.Application package MediatR
dotnet add src/{SolutionName}.Application package AutoMapper
dotnet add src/{SolutionName}.Application package FluentValidation
dotnet add src/{SolutionName}.Application package FluentValidation.DependencyInjectionExtensions
dotnet add src/{SolutionName}.Application package Microsoft.EntityFrameworkCore
```

> **Note on EF Core in Application:**  
> `Microsoft.EntityFrameworkCore` may be included in Application to support Linq abstractions (e.g. `IQueryable`, async EF LINQ operators, or generic repository query contracts). Alternatively, keep pure abstractions without direct provider ties.

Do NOT hard-code old versions.

Use versions compatible with the detected .NET SDK.

---

# 12. MediatR / CQRS Configuration

Create:

```text
src/{SolutionName}.Application/DependencyInjection.cs
```

Expose:

```csharp
AddApplication()
```

Configure MediatR to scan the Application assembly.

Use the registration API supported by the installed MediatR version.

Do NOT create:

- Commands
- Queries
- Handlers
- Notifications
- Business pipeline behaviors

The configuration must allow future features to be automatically
discovered.

---

# 13. FluentValidation Configuration

Register FluentValidation from the Application assembly.

Use the current API supported by the installed version.

Do NOT create business validators.

Future validators must be discoverable without modifying the central
Application registration.

Create:

```text
Common/Behaviors/
```

as an architectural placeholder.

Do not create a validation pipeline behavior unless required by the
selected implementation.

---

# 14. AutoMapper Configuration

Register AutoMapper by scanning the Application assembly.

Create:

```text
Mappings/
```

Do NOT create business mapping profiles.

Future profiles must be automatically discoverable.

---

# 15. Application Dependency Injection

`DependencyInjection.cs` must provide:

```csharp
public static IServiceCollection AddApplication(
    this IServiceCollection services)
```

It must register:

- MediatR
- AutoMapper
- FluentValidation

Do not register these packages directly in API `Program.cs`.

---

# 16. Infrastructure Layer

Project:

```text
{SolutionName}.Infrastructure
```

References:

```text
Application
Domain
```

This layer contains infrastructure implementations.

Create:

```text
Persistence/
Services/
Common/
```

This is intentional:

> Persistence is a concern of Infrastructure and is NOT a separate
> project.

Remove the default class-library sample file.

Create:

```text
DependencyInjection.cs
```

Expose:

```csharp
AddInfrastructure(...)
```

---

# 17. Persistence Structure & Data Access Layer Folders

Persistence MUST exist inside Infrastructure:

```text
Infrastructure/
└── Persistence/
    ├── Context/
    ├── Configurations/
    ├── Repositories/
    │   └── Common/
    └── Migrations/
```

The Data Access Layer requires standard path segregation:
- `Context/`: Houses the ApplicationDbContext and EF Core context configurations.
- `Configurations/`: Houses entity type configurations (`IEntityTypeConfiguration<T>`).
- `Repositories/`: Houses specific and concrete repository implementations.
- `Repositories/Common/`: Houses base repository implementations (such as generic repository base classes, e.g., `GenericRepository<T>`, and `UnitOfWork` implementations).
- `Migrations/`: Reserved for generated EF Core schema migrations.

Do NOT create a separate:

```text
{SolutionName}.Persistence
```

project.

Do NOT place Persistence under Application.

Persistence is an infrastructure implementation detail.

---

# 18. Entity Framework Core & Repository Packages

Install:

```powershell
dotnet add src/{SolutionName}.Infrastructure package Microsoft.EntityFrameworkCore
dotnet add src/{SolutionName}.Infrastructure package Microsoft.EntityFrameworkCore.Design
dotnet add src/{SolutionName}.Infrastructure package Microsoft.EntityFrameworkCore.Relational
```

Install tool/design support in the API project so EF Core CLI tools (`dotnet ef migrations add`, `dotnet ef database update`) function properly with the startup project:

```powershell
dotnet add src/{SolutionName}.API package Microsoft.EntityFrameworkCore.Design
```

Do NOT automatically install a specific database provider.

Do not assume:

- SQL Server
- PostgreSQL
- MySQL
- SQLite

The database provider must be selected based on actual project
requirements.

Do NOT create:

- DbSets for business entities
- Business repositories
- Entity configurations
- Migrations
- Seed data

The Persistence directories should remain empty architectural
placeholders ready for repository and context implementation unless generic foundational repository classes are explicitly requested.

---

# 19. Infrastructure Dependency Injection

Create:

```text
Infrastructure/DependencyInjection.cs
```

Expose:

```csharp
AddInfrastructure(
    IServiceCollection services,
    IConfiguration configuration)
```

or the appropriate equivalent.

Infrastructure registration should be responsible for:

- Persistence & DbContext registration
- Repository bindings (registering repository implementations to their respective interfaces)
- External services
- Infrastructure implementations

Do not put infrastructure registration in Application.

---

# 20. API Project

Project:

```text
{SolutionName}.API
```

Created using:

```powershell
dotnet new webapi
```

References:

```text
Application
Infrastructure
```

The API is the backend composition root.

The API is responsible for:

- HTTP endpoints
- Controllers
- Request/response models where appropriate
- API configuration
- Authentication/authorization foundation if later required
- Dependency injection composition
- Middleware
- Swagger/OpenAPI

Do NOT create business endpoints.

Do NOT create sample CRUD controllers.

Do NOT create sample WeatherForecast functionality.

Remove template-generated business/demo API files.

---

# 21. API Dependency Injection

`Program.cs` must act as the backend composition root.

Conceptually:

```csharp
builder.Services.AddApplication();

builder.Services.AddInfrastructure(
    builder.Configuration);
```

Use the exact signatures required by the generated implementation.

Do not directly configure MediatR, AutoMapper, or FluentValidation
inside API `Program.cs`.

Those belong to Application.

Do not directly configure EF Core or individual repositories inside API `Program.cs`.

That belongs to Infrastructure.

---

# 22. API Foundation

The API must remain a valid ASP.NET Core Web API.

Preserve the necessary:

```text
Controllers/
Program.cs
```

OpenAPI/Swagger support may remain enabled as part of the standard API
foundation.

Do not create actual business controllers.

The API must compile and be runnable.

---

# 23. Web Project

Project:

```text
{SolutionName}.Web
```

Created using:

```powershell
dotnet new mvc
```

The Web project is the presentation layer.

It uses:

```text
ASP.NET Core MVC
Razor Views
.cshtml
Controllers
ViewModels
```

Do NOT install:

- Angular
- React
- Vue
- Blazor
- SPA frameworks

The Web project should NOT reference:

```text
Domain
Application
Infrastructure
API
```

The Web application communicates with the API using HTTP.

---

# 24. Web API Client Foundation

The Web project should be prepared for API communication.

Create an appropriate placeholder structure such as:

```text
Services/
    Api/
```

or:

```text
Clients/
```

Do NOT create business-specific API clients.

Do NOT create:

```text
ProductApiClient
UserApiClient
OrderApiClient
```

The goal is only to establish the architectural location for future
typed HTTP clients.

Use built-in `HttpClient` infrastructure.

Do not introduce Refit or another third-party HTTP client library
unless explicitly requested.

---

# 25. Web Dependency Injection

The Web project's `Program.cs` should configure MVC normally.

If an empty generic API client infrastructure is required, configure
`HttpClient` using the standard ASP.NET Core mechanisms.

Do NOT register Application services directly in Web.

Do NOT register Infrastructure services directly in Web.

The Web application is a client of the API.

---

# 26. API vs Web Responsibility

Maintain this strict separation:

### API

```text
HTTP
 ↓
Controller
 ↓
MediatR
 ↓
Application Handler
 ↓
Repository / DbContext / Infrastructure
```

### Web

```text
Browser
 ↓
MVC Controller
 ↓
API Client / HttpClient
 ↓
HTTP
 ↓
API
```

The Web MVC controller must NOT directly invoke:

```text
MediatR
DbContext
Repository
Domain service
Infrastructure service
```

It should communicate with the backend through the API.

---

# 27. Final Directory Structure

The expected structure is:

```text
{SolutionName}/
│
├── src/
│   │
│   ├── {SolutionName}.Domain/
│   │   ├── Entities/
│   │   ├── Common/
│   │   ├── Enums/
│   │   └── Repositories/
│   │
│   ├── {SolutionName}.Application/
│   │   ├── Common/
│   │   │   ├── Interfaces/
│   │   │   │   ├── Persistence/
│   │   │   │   └── Repositories/
│   │   │   ├── Models/
│   │   │   └── Behaviors/
│   │   ├── Features/
│   │   ├── Mappings/
│   │   └── DependencyInjection.cs
│   │
│   ├── {SolutionName}.Infrastructure/
│   │   ├── Persistence/
│   │   │   ├── Context/
│   │   │   ├── Configurations/
│   │   │   ├── Repositories/
│   │   │   │   └── Common/
│   │   │   └── Migrations/
│   │   ├── Services/
│   │   ├── Common/
│   │   └── DependencyInjection.cs
│   │
│   ├── {SolutionName}.API/
│   │   ├── Controllers/
│   │   └── Program.cs
│   │
│   └── {SolutionName}.Web/
│       ├── Controllers/
│       ├── Models/
│       ├── Services/
│       │   └── Api/
│       ├── Views/
│       ├── wwwroot/
│       └── Program.cs
│
└── {SolutionName}.sln
```

---

# 28. Windows Compatibility

This skill must work in:

- Windows PowerShell
- Windows Terminal
- VS Code integrated terminal
- Claude Code on Windows

Prefer cross-platform `dotnet` CLI commands.

Do NOT depend on:

- Bash
- WSL
- Linux-specific commands
- `rm`
- `mkdir -p`
- `grep`
- `sed`
- Unix-specific environment variables

Use `dotnet` CLI commands and file operations where appropriate.

Use relative paths such as:

```text
src/{SolutionName}.Domain
```

Do not assume a Unix filesystem.

---

# 29. Package Compatibility

Never copy package versions from old tutorials.

Determine:

1. Installed .NET SDK.
2. Target framework.
3. Latest stable compatible versions.
4. Install packages.
5. Inspect generated `.csproj`.
6. Restore.
7. Build.

If a package's newest release is incompatible with the selected target
framework, select the newest compatible stable release.

Do not downgrade the .NET SDK simply to accommodate an outdated package.

---

# 30. Verification

After scaffolding:

### Restore

```powershell
dotnet restore
```

### Build

```powershell
dotnet build
```

The solution MUST build successfully.

If it fails:

1. Read the actual error.
2. Diagnose the cause.
3. Correct the generated solution.
4. Restore again.
5. Build again.
6. Continue until successful or a genuine environment limitation
   prevents completion.

Do not merely report a build error without attempting to resolve it.

---

# 31. Verify Project Count

Run:

```powershell
dotnet sln list
```

Verify exactly five projects:

```text
Domain
Application
Infrastructure
API
Web
```

There must be no Persistence project.

---

# 32. Verify Project References

Verify:

```text
Domain
  → none

Application
  → Domain

Infrastructure
  → Application
  → Domain

API
  → Application
  → Infrastructure

Web
  → none
```

The Web project must not reference the backend projects.

---

# 33. Verify Packages

Application:

```text
MediatR
AutoMapper
FluentValidation
FluentValidation.DependencyInjectionExtensions
Microsoft.EntityFrameworkCore
```

Infrastructure:

```text
Microsoft.EntityFrameworkCore
Microsoft.EntityFrameworkCore.Design
Microsoft.EntityFrameworkCore.Relational
```

API:

```text
Microsoft.EntityFrameworkCore.Design
```

No database provider should be installed unless explicitly requested.

---

# 34. Verify Business Feature Absence

Confirm that the scaffold does NOT contain:

- Business entities
- Commands
- Queries
- Handlers
- DTOs
- Validators
- Mapping profiles
- Repositories for business entities
- Controllers for business entities
- Migrations
- Seed data
- CRUD examples
- WeatherForecast demo

The scaffold must contain architecture only.

---

# 35. Final Report

After successful completion, report:

### Solution

```text
{SolutionName}
```

### SDK

```text
Installed SDK:
Target Framework:
```

### Projects

```text
1. Domain
2. Application
3. Infrastructure
4. API
5. Web
```

### Architecture

```text
Domain → Application → Infrastructure → API
Web → HTTP → API
```

### CQRS

```text
MediatR: Configured
CQRS foundation: Ready
Business handlers: None
```

### Mapping

```text
AutoMapper: Configured
Business profiles: None
```

### Validation

```text
FluentValidation: Configured
Business validators: None
```

### Persistence & Data Access Layer

```text
Location: Infrastructure/Persistence
EF Core: Installed (Core, Design, Relational)
Design Tools: Installed in API (Startup project)
Repository Paths:
  - Interfaces: Application/Common/Interfaces/Repositories & Persistence
  - Implementations: Infrastructure/Persistence/Repositories (with Common/ for base classes)
  - DbContext & Config: Infrastructure/Persistence/Context & Configurations
  - Migrations: Infrastructure/Persistence/Migrations
Database provider: Not selected
Migrations: None
```

### Web

```text
ASP.NET Core MVC: Configured
Razor Views: Enabled
API communication: Prepared
```

### Verification

```text
Project count: 5
Project references: Verified
dotnet restore: Successful
dotnet build: Successful
```

Explicitly state:

> The solution contains architectural and technical foundations only.
> No business features or use cases were generated.