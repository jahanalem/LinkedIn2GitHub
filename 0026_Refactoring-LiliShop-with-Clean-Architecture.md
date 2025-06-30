# From Monolith to Masterpiece: Refactoring LiliShop with Clean Architecture

Welcome to the technical overview of LiliShop, a modern e-commerce platform built with .NET 9 and Angular 20. This article documents the journey of refactoring the LiliShop backend from a traditional layered architecture to a more robust, maintainable, and scalable setup using Clean Architecture.

This document will guide you through the "what," the "why," and the "how" of this transition, providing a clear blueprint of the new project structure.

<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [From Monolith to Masterpiece: Refactoring LiliShop with Clean Architecture](#from-monolith-to-masterpiece-refactoring-lilishop-with-clean-architecture)
   * [1\. What is Clean Architecture?](#1-what-is-clean-architecture)
   * [2\. Why Migrate LiliShop to Clean Architecture?](#2-why-migrate-lilishop-to-clean-architecture)
   * [3\. The Structure of LiliShop in Clean Architecture](#3-the-structure-of-lilishop-in-clean-architecture)
      + [3.1. The Core: `LiliShop.Domain`](#31-the-core-lilishopdomain)
      + [3.2. The Use Cases: `LiliShop.Application`](#32-the-use-cases-lilishopapplication)
      + [3.3. The Details: `LiliShop.Infrastructure`](#33-the-details-lilishopinfrastructure)
      + [3.4. The Entry Point: `LiliShop.API`](#34-the-entry-point-lilishopapi)
   * [4\. The Dependency Rule in Action](#4-the-dependency-rule-in-action)
   * [5\. Overcoming a Migration Challenge: Decoupling Identity](#5-overcoming-a-migration-challenge-decoupling-identity)
   * [6. Code Metrics: The Proof in Numbers](#6-code-metrics-the-proof-in-numbers)
   * [Conclusion](#conclusion)

<!-- TOC end -->

## 1\. What is Clean Architecture?

Clean Architecture, a concept popularized by Robert C. Martin ("Uncle Bob"), is a software design philosophy that emphasizes the separation of concerns. Its primary goal is to create systems that are:

  * **Independent of Frameworks:** The core business logic doesn't depend on a specific web framework or library.
  * **Testable:** Business rules can be tested without the UI, database, or any external element.
  * **Independent of UI:** The UI can change easily without changing the rest of the system.
  * **Independent of Database:** The core business rules are not tied to a specific database technology like SQL Server or any other.

The fundamental rule of Clean Architecture is the **Dependency Rule**. All source code dependencies must point inwards, from outer layers to inner layers. The inner layers define policies and abstractions (interfaces), while the outer layers implement the details.

## 2\. Why Migrate LiliShop to Clean Architecture?

The original version of LiliShop used a traditional N-Layer architecture (Model, DataAccess, Service, API). While functional, this approach often leads to a few common problems as a project grows:

  * **Tight Coupling:** Business logic often becomes entangled with data access code (like Entity Framework) or external services.
  * **Difficult to Test:** Testing core business rules required mocking the database and other infrastructure, making tests complex.
  * **Hard to Maintain:** Changes to one part of the system, like the database, could cause a ripple effect of changes throughout the application.

Migrating to Clean Architecture was a strategic decision to build a more professional and long-lasting application. The key motivators were:

  * **Enhanced Maintainability:** By separating concerns into distinct layers, finding and modifying code becomes much simpler.
  * **Improved Testability:** The core business logic in the `Domain` and `Application` layers can be tested in complete isolation.
  * **Technology Independence:** The architecture makes it easier to swap out external dependencies. For example, we could replace Stripe with another payment processor or SQL Server with another database with minimal impact on the core application.

## 3\. The Structure of LiliShop in Clean Architecture

LiliShop is now organized into four primary projects, each representing a layer in the Clean Architecture style: `Domain`, `Application`, `Infrastructure`, and `API`.

```
/LiliShop
|-- LiliShop.Domain/
|-- LiliShop.Application/
|-- LiliShop.Infrastructure/
`-- LiliShop.API/
```
![Clean-Architecture](https://github.com/user-attachments/assets/5b8aff8f-4c0c-480f-ba83-423082db4b52)


### 3.1. The Core: `LiliShop.Domain`

The `Domain` layer is the heart of the application. It contains the enterprise-wide business rules and logic. It is the most independent layer and has no dependencies on any other project in the solution.

  * **Role:** To define the business objects (Entities), their rules, and core interfaces.
  * **What Belongs Here:**
      * `Entities`: Core business models like `Product`, `Brand`, `Order`, and `NotificationSubscription`. These are plain C\# objects.
      * `Enums` & `Constants`: Business-specific enumerations and constant values.
      * `Specifications`: The `ISpecification<T>` interface, which defines the contract for building query specifications in a database-agnostic way.
      * `PrintessEditor`: Data models specific to the Printess Editor feature integration.
  * **What to Avoid:**
      * Any code related to databases (e.g., Entity Framework Core).
      * Dependencies on any external libraries or frameworks (like `Microsoft.AspNetCore.Identity`).
      * Logic that depends on external services like email or payment providers.

Here is the project file for `LiliShop.Domain`, demonstrating its lack of dependencies. Note the `<DisableImplicitNamespaceImports>` tag, which explicitly prevents accidental dependencies on frameworks like Entity Framework Core.

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <DisableImplicitNamespaceImports>Microsoft.EntityFrameworkCore;Microsoft.AspNetCore.Identity</DisableImplicitNamespaceImports>
  </PropertyGroup>
</Project>
```

### 3.2. The Use Cases: `LiliShop.Application`

The `Application` layer orchestrates the data flow using the core entities and rules from the `Domain` layer. It contains application-specific business logic and defines the interfaces that are implemented by the outer layers.

  * **Role:** To execute the use cases of the application (e.g., "get a list of products," "create an order").
  * **What Belongs Here:**
      * `DTOs`: Data Transfer Objects used to shape the data sent to and from the API layer, preventing the exposure of `Domain` entities directly.
      * `Interfaces`: This is the most critical part. It defines contracts for infrastructure concerns.
          * `IRepositories`: `IUnitOfWork`, `IGenericRepository<T>`.
          * `IShopDbContext`: An abstraction for the database context.
          * `External Services`: `IEmailService`, `IPhotoService`, `IPrintessService`. The application layer dictates what these services *do*, not *how* they do it.
      * `Mappings`: Contains mapping logic, including AutoMapper profiles and custom mapping functions, to convert `Domain` entities to `DTOs` and vice-versa.
      * `Specifications`: Implementations of the specification pattern (`BaseSpecification<T>`) used to build query logic that will be executed by the Infrastructure layer.
  * **What to Avoid:**
      * Direct implementation of data access or other infrastructure concerns.
      * Code that deals with HTTP, web sockets, or any presentation-layer technology.
      * Direct dependencies on concrete implementations like `SendGrid` or `Cloudinary`.

The `LiliShop.Application` project only has a dependency on `LiliShop.Domain`.

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
  <ItemGroup>
    <ProjectReference Include="..\LiliShop.Domain\LiliShop.Domain.csproj" />
  </ItemGroup>
</Project>
```

### 3.3. The Details: `LiliShop.Infrastructure`

The `Infrastructure` layer is where the "details" live. It provides the concrete implementations for the interfaces defined in the `Application` layer. This is where the application interacts with the outside world.

  * **Role:** To implement data persistence, interact with third-party services, and handle other external concerns.
  * **What Belongs Here:**
      * `Data`: The implementation of `IShopDbContext` using `Microsoft SQL Server` with `Entity Framework Core`. This includes migrations, database seeding, and the concrete implementation of repositories.
      * `Services`: The concrete implementations of external services. For example, `EmailService` uses `SendGrid`, and `PhotoService` uses `Cloudinary`.
      * `Identity`: All implementation details for user authentication and authorization using `Microsoft.AspNetCore.Identity`. The `ApplicationUser` class lives here.
      * `Resources`: Contains files like email HTML templates.
  * **What to Avoid:**
      * Business logic. This layer should be "dumb" and only contain the code necessary to implement the interfaces from the `Application` layer.

The `LiliShop.Infrastructure` project depends on both `LiliShop.Application` (to implement its interfaces) and `LiliShop.Domain`. This is where all the external NuGet packages for databases, email, payments, etc., are referenced.

```xml
<Project Sdk="Microsoft.NET.Sdk">
	<PropertyGroup>
		<TargetFramework>net9.0</TargetFramework>
		<ImplicitUsings>enable</ImplicitUsings>
		<Nullable>enable</Nullable>
	</PropertyGroup>
	<ItemGroup>
		<PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="9.0.5" />
		<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="9.0.5" />
		<PackageReference Include="SendGrid" Version="9.29.3" />
		<PackageReference Include="CloudinaryDotNet" Version="1.27.5" />
		<PackageReference Include="Stripe.net" Version="48.2.0" />
		</ItemGroup>
	<ItemGroup>
		<ProjectReference Include="..\LiliShop.Application\LiliShop.Application.csproj" />
		<ProjectReference Include="..\LiliShop.Domain\LiliShop.Domain.csproj" />
	</ItemGroup>
</Project>
```

### 3.4. The Entry Point: `LiliShop.API`

The `API` layer is the entry point to the system. For LiliShop, this is an ASP.NET Core Web API project. It's a thin layer responsible for handling HTTP requests and forwarding them to the `Application` layer.

  * **Role:** To expose the application's use cases to the outside world (in this case, our Angular frontend).
  * **What Belongs Here:**
      * `Controllers`: API endpoints that receive requests, pass them to the `Application` layer, and return responses.
      * `Middleware`: Custom middleware for logging, error handling, etc.
      * `Program.cs`: The composition root, where all services and dependencies from all layers are registered with the Dependency Injection container.
      * Configuration files like `appsettings.json` and `nlog.config`.
  * **What to Avoid:**
      * Business logic. Controllers should be lean and only contain logic related to handling the HTTP interaction.

The `LiliShop.API` project references `LiliShop.Application` (to call its services) and `LiliShop.Infrastructure` (to configure dependency injection).

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  <ItemGroup>
    <ProjectReference Include="..\LiliShop.Application\LiliShop.Application.csproj" />
    <ProjectReference Include="..\LiliShop.Infrastructure\LiliShop.Infrastructure.csproj" />
  </ItemGroup>
</Project>
```

## 4\. The Dependency Rule in Action

The relationship between the layers strictly follows the Dependency Rule. Dependencies flow inwards, ensuring that the core business logic remains pure and independent.

`API` -\> `Application` -\> `Domain`

The `Infrastructure` layer also depends on the `Application` layer (to implement its interfaces), effectively "plugging into" the application's core logic. This is known as Dependency Inversion.

## 5\. Overcoming a Migration Challenge: Decoupling Identity

One of the main challenges during the refactoring was decoupling `Microsoft.AspNetCore.Identity` from the core domain.

  * **The Problem:** In the previous architecture, the `ApplicationUser` class (from Identity) was likely treated as a core model. A domain entity like `NotificationSubscription` might have had a direct navigation property to `ApplicationUser`. This created a direct dependency from the business domain to a specific infrastructure framework, violating a key principle of Clean Architecture.

  * **The Clean Architecture Solution:**

    1.  The `ApplicationUser` entity, being part of an external framework, was moved to the `LiliShop.Infrastructure` project where it belongs.
    2.  The `NotificationSubscription` entity in `LiliShop.Domain` was modified. Instead of holding a reference to the `ApplicationUser` object, it now only stores the `UserId` (e.g., an `int` or `string`).
    3.  When fetching data for a user-facing feature, such as viewing all price drop subscriptions, the query is now performed outside the domain. A service in the `Application` or `Infrastructure` layer joins the `NotificationSubscriptions` table with the `Users` table to create a DTO (Data Transfer Object) with all the required information.

Here is an example of how this is handled now. This method, residing in a service within the `Infrastructure` layer, can safely query across both tables to construct a view model without the `Domain` layer ever knowing about the `ApplicationUser` entity.

```csharp
// This query now happens in a layer that is allowed to know about both
// NotificationSubscription (Domain) and ApplicationUser (Infrastructure).
public async Task<List<PriceDropSubscriptionDetailsDto>> GetPriceDropSubscriptionsAsync()
{
    return await _context.NotificationSubscriptions
        .Where(s => s.AlertType == AlertType.PriceDrop)
        .Join(_context.Users,
            s => s.UserId,
            u => u.Id,
            (s, u) => new PriceDropSubscriptionDetailsDto
            {
                UserId = u.Id,
                DisplayName = u.UserName,
                Email = u.Email,
                ProductId = s.ProductId.Value,
                ProductName = s.Product.Name,
                ProductPrice = s.Product.Price,
                SubscriptionDate = s.CreatedDate.Value,
                PreviousPrice = s.Product.PreviousPrice ?? 0m
            })
        .ToListAsync();
}
```

This approach correctly separates the domain model from the infrastructure implementation, making the system much cleaner and more aligned with architectural principles.

## 6. Code Metrics: The Proof in Numbers

Before concluding, let's validate the refactor's impact with hard metrics. Here are the key takeaways from our code analysis (excluding auto-generated migration files):

![code-metrics-results-02](https://github.com/user-attachments/assets/37061ac8-13f4-4477-ad09-3875ef2c0bc3)

![code-metrics-results-01](https://github.com/user-attachments/assets/d67c812d-771e-4c4f-8350-fdd472e13010)



## Conclusion

Refactoring LiliShop to Clean Architecture was a significant investment, but one that provides immense value. The resulting application is far more organized, maintainable, testable, and flexible. This solid architectural foundation will make it easier to add new features and adapt to changing technologies in the future, ensuring LiliShop remains a robust and high-quality e-commerce platform.

#LiliShop, #CleanArchitecture, #DotNet, #Angular, #Refactoring, #SoftwareArchitecture, #CSharp, #SOLIDPrinciples, #WebDevelopment
