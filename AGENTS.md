# AGENTS.md

## Purpose

This document defines coding standards, architectural guidance, and expectations for all contributors and AI coding agents working on C# and ASP.NET Core projects.

When multiple valid approaches exist, choose the simplest solution that preserves readability and maintainability.

---

# Table of Contents

## Part I: C# Language & Coding Standards

- [Core Principles](#core-principles)
- [C# Standards](#c-standards)
  - [Naming](#naming)
  - [Language Features](#language-features)
  - [Async](#async)
  - [Exceptions](#exceptions)
  - [LINQ](#linq)
- [Documentation Standards](#documentation-standards)
- [Constants](#constants)
- [Options Pattern](#options-pattern)
- [Preferred Framework Patterns](#preferred-framework-patterns)
- [Domain Behaviour Pattern](#domain-behaviour-pattern)

## Part II: ASP.NET Core & System Architecture

- [Architecture](#architecture)
  - [General](#general)
  - [Messaging Infrastructure](#messaging-infrastructure)
- [Service Naming Conventions](#service-naming-conventions)
- [Event-Driven Architecture](#event-driven-architecture)
- [Service-Oriented Architecture](#service-oriented-architecture)
- [Domain-Driven Design](#domain-driven-design)
- [Resilience and Reliability](#resilience-and-reliability)
- [ASP.NET Core Standards](#aspnet-core-standards)
  - [API Style](#api-style)
  - [Endpoint Design](#endpoint-design)
  - [Dependency Injection](#dependency-injection)
  - [Configuration](#configuration)
  - [Logging](#logging)
- [Persistence Standards](#persistence-standards)
- [Testing Standards](#testing-standards)
- [Security Standards](#security-standards)
- [Problem Details](#problem-details)
- [AI Agent Instructions](#ai-agent-instructions)

---

# Core Principles

1. Prefer simplicity over cleverness.
2. Follow SOLID principles.
3. Avoid duplication (DRY).
4. Avoid unnecessary abstractions (YAGNI).
5. Favour readability over brevity.
6. Keep classes cohesive.
7. Keep methods focused.
8. Prefer explicit code over framework magic.

---

# C# Standards

## Naming

### Types

Use PascalCase.

```csharp
public class UserService
{
}
```

### Methods

Use PascalCase and verb-based names.

```csharp
CreateUser()
ValidateToken()
CalculateTotal()
```

### Properties

Use PascalCase.

Boolean properties should be predicates.

```csharp
IsActive
HasPermission
CanDelete
```

### Interfaces

Prefix with I.

```csharp
IUserRepository
IEmailSender
```

### Fields

Use _camelCase for private fields.

```csharp
private readonly ILogger<UserService> _logger;
```

### Parameters and Locals

Use camelCase.

---

## Language Features

### Namespaces

Use file-scoped namespaces.

```csharp
namespace MyApplication.Domain;
```

### var

Use var when the type is obvious.

```csharp
var user = new User();
```

Prefer explicit types for primitives.

```csharp
int count = 10;
bool enabled = true;
```

### Records

Prefer records for DTOs and immutable data.

```csharp
public sealed record CreateUserRequest(
    string Email,
    string Password);
```

### Required Members

Use required properties where appropriate.

### Primary Constructors

Use when they improve readability.

Avoid excessive constructor parameter lists.

### Collection Expressions

Prefer modern collection syntax.

```csharp
List<string> names = [];
```

### Pattern Matching

Prefer modern pattern matching.

```csharp
if (user is null)
{
}
```

---

## Async

Use asynchronous APIs for I/O operations.

Avoid:

```csharp
.Result
.Wait()
.GetAwaiter().GetResult()
```

Prefer:

```csharp
await repository.SaveAsync();
```

---

## Exceptions

Exceptions are for exceptional situations.

Do not use exceptions for normal control flow.

Prefer explicit result types for expected failures.

---

## LINQ

Use LINQ when it improves readability.

Avoid overly complex query chains.

Break large queries into smaller named steps.

---

# Documentation Standards

## XML Documentation

Use XML documentation comments exclusively for public APIs and important domain behaviour.

Prefer:

```csharp
/// <summary>
/// Creates a new user account.
/// </summary>
/// <param name="request">The creation request.</param>
/// <returns>The created user.</returns>
```

Avoid:

```csharp
// Creates a user
```

Use XML documentation for:

- All types
- All methods
- Domain concepts
- Complex business rules

Avoid non-XML comments unless absolutely necessary.

---

# Constants

Avoid magic strings and magic numbers.

Prefer constants abstractions.

```csharp
public static class SessionStatus
{
    public const string Active = "Active";
    public const string Revoked = "Revoked";
    public const string Expired = "Expired";
}
```

```csharp
public static class ValidationLimits
{
    public const int MaxEmailLength = 256;
    public const int MaxDisplayNameLength = 100;
}
```

Guidelines:

- Prefer static classes containing const values.
- Centralize domain constants.
- Avoid repeating string literals.
- Avoid unexplained numeric values.
- Give constants meaningful names.

---

# Options Pattern

Prefer strongly typed options classes.

Avoid reading configuration values directly throughout the application.

Prefer dedicated ConfigureOptions classes using IConfigureOptions<T>.

Example:

```csharp
public sealed class JwtOptions
{
    public required string Issuer { get; init; }
    public required string Audience { get; init; }
}
```

```csharp
public sealed class ConfigureJwtOptions
    : IConfigureOptions<JwtOptions>
{
    private readonly IConfiguration _configuration;

    /// <summary>
    /// Initializes a new instance of the ConfigureJwtOptions class.
    /// </summary>
    /// <param name="configuration">Application configuration.</param>
    public ConfigureJwtOptions(
        IConfiguration configuration)
    {
        _configuration = configuration;
    }

    /// <summary>
    /// Configures JWT options.
    /// </summary>
    /// <param name="options">The options instance.</param>
    public void Configure(JwtOptions options)
    {
        _configuration
            .GetSection("Jwt")
            .Bind(options);
    }
}
```

Registration:

```csharp
services.AddSingleton<
    IConfigureOptions<JwtOptions>,
    ConfigureJwtOptions>();
```

Avoid:

```csharp
services.Configure<JwtOptions>(
    configuration.GetSection("Jwt"));
```

Guidelines:

- Prefer IConfigureOptions<T> implementations.
- Keep configuration binding in a single location.
- Prefer strongly typed options.
- Avoid magic configuration keys throughout the application.
- Inject IOptions<T>, IOptionsSnapshot<T>, or IOptionsMonitor<T>.
- Keep Program.cs focused on composition rather than implementation details.

---

# Problem Details

Prefer extending problem details as per below:

```csharp
/// <summary>
/// Extension methods for configuring ProblemDetails with custom properties.
/// </summary>
public static class ProblemDetailsExtensions
{
    /// <summary>
    /// Adds ProblemDetails services with custom extensions for detailed error information.
    /// </summary>
    /// <param name="services">The service collection to add ProblemDetails services to.</param>
    /// <returns>The service collection for method chaining.</returns>
    public static IServiceCollection AddCustomProblemDetails(this IServiceCollection services)
    {
        services.AddProblemDetails();

        services.ConfigureOptions<ConfigureProblemDetailsOptions>();

        return services;
    }
}
```

```csharp
/// <summary>
/// Configures ProblemDetails options with custom extensions for detailed error information.
/// Adds contextual metadata to error responses including machine name, timestamps,
/// request identifiers, exception details, and user information for comprehensive debugging.
/// </summary>
public sealed class ConfigureProblemDetailsOptions : IConfigureOptions<ProblemDetailsOptions>
{
    /// <inheritdoc/>
    public void Configure(ProblemDetailsOptions options)
    {
        options.CustomizeProblemDetails = context =>
        {
            // Machine name where the error occurred for infrastructure tracking
            context.ProblemDetails.Extensions["machineName"] = Environment.MachineName;

            // UTC timestamp when the error occurred for temporal correlation
            context.ProblemDetails.Extensions["timestamp"] = DateTime.UtcNow;

            // Request identifier for distributed tracing and log correlation
            context.ProblemDetails.Extensions["requestId"] = context.HttpContext.TraceIdentifier;

            // Service name for identifying the source service in distributed systems
            context.ProblemDetails.Extensions["serviceName"] = ServiceDefaults.Name;

            // User agent string for client identification and debugging
            context.ProblemDetails.Extensions["userAgent"] = context.HttpContext.Request.Headers.UserAgent.ToString();

            // HTTP method used in the request for context and debugging
            context.ProblemDetails.Extensions["method"] = context.HttpContext.Request.Method;

            // Exception type and stack trace when an exception caused the error
            if (context.Exception is not null)
            {
                // Full type name of the exception for debugging and error categorization
                context.ProblemDetails.Extensions["exceptionType"] = context.Exception.GetType().FullName;

                // Stack trace for detailed debugging (should be filtered in production)
                context.ProblemDetails.Extensions["stackTrace"] = context.Exception.StackTrace ?? string.Empty;
            }

            // User identifier if the request was authenticated
            var userId = context.HttpContext.User?.FindFirst("sub")?.Value
                         ?? context.HttpContext.User?.FindFirst("userId")?.Value;
            if (!string.IsNullOrEmpty(userId))
            {
                context.ProblemDetails.Extensions["userId"] = userId;
            }
        };
    }
}
```

---

# Preferred Framework Patterns

Prefer:

- Extension methods for entity behaviour.
- Extension methods for IServiceCollection registrations.
- Extension methods for endpoint mapping.
- ConfigureOptions classes for option binding.
- XML documentation comments.
- Constants classes over magic strings and magic numbers.
- Strongly typed options.

---

# Domain Behaviour Pattern

## Entity Behaviour

Prefer adding behaviour through extension methods rather than methods on entity classes.

Example:

```csharp
/// <summary>
/// Extension methods for managing Session entity properties and status.
/// </summary>
public static class SessionExtensions
{
    /// <summary>
    /// Marks the session as revoked.
    /// </summary>
    /// <param name="session">The session to revoke.</param>
    /// <param name="revokedAt">The timestamp when revoked. Defaults to current UTC time.</param>
    /// <returns>The session for method chaining.</returns>
    public static Session MarkAsRevoked(
        this Session session,
        DateTimeOffset? revokedAt = null)
    {
        session.Status = SessionStatus.Revoked;
        session.RevokedAt = revokedAt ?? DateTimeOffset.UtcNow;

        return session;
    }
}
```

Guidelines:

- Prefer extension methods for entity behaviour.
- Keep entities focused on state and persistence concerns.
- Group extensions by entity.
- Return the entity when method chaining improves readability.
- Place extensions close to the feature they belong to.

---

# Architecture

## General

- Prefer Vertical Slice Architecture.
- Organize code by feature rather than technical concern.
- Prefer folders over additional assemblies unless there is a demonstrated need.
- Keep dependency direction flowing inward.

Preferred structure:

```text
src/
├── Api/
├── Application/
├── Domain/
├── Persistence/
└── Infrastructure/

tests/
```

For small and medium-sized applications, implement layers as folders within a single project unless separation into assemblies provides a clear benefit.

## Messaging Infrastructure

- Use the Outbox Pattern to ensure atomicity between database commits and event publishing.
- Prefer a centralized messaging abstraction (e.g., `IEventPublisher`, `IEventConsumer`) over direct broker SDK usage.
- Keep broker-specific code isolated in the Infrastructure layer.

Preferred structure for event-driven services:

```text
src/
├── Api/
├── Application/
│   ├── Features/
│   └── Events/
├── Domain/
│   └── Events/
├── Persistence/
│   └── Outbox/
└── Infrastructure/
    ├── Messaging/
    │   ├── Publishers/
    │   ├── Subscribers/
    │   ├── Serializers/
    │   └── Outbox/
    └── ...

tests/
```

---

# Service Naming Conventions

## Assembly Naming

- All service assemblies must include `.Service.` in their name.
- Use the format: `{Company}.{Domain}.Service.{Capability}`

Examples:

```csharp
ArbitrageIQ.Trading.Service.OrderManagement
ArbitrageIQ.Trading.Service.Pricing
ArbitrageIQ.Trading.Service.Execution
```

## Namespace Naming

- Root namespaces must match the assembly name.
- Use file-scoped namespaces as per C# standards.

```csharp
namespace ArbitrageIQ.Trading.Service.OrderManagement.Domain;
```

## Project Naming

- Project names (`.csproj`) must follow assembly naming conventions.
- Solution folders should group services by domain or bounded context.

## Repository Naming

- Git repositories should follow the pattern: `{domain}-service-{capability}`
- Use lowercase with hyphen separators.

Examples:

```text
trading-service-order-management
trading-service-pricing
trading-service-execution
```

## Service Identifier

- Each service must define a stable, unique service name constant.
- Use this identifier for logging, tracing, and service discovery.

```csharp
public static class ServiceDefaults
{
    public const string Name = "order-management-service";
}
```

---

# Event-Driven Architecture

## Communication Style

- Prefer asynchronous messaging over synchronous HTTP calls between services.
- Use synchronous HTTP only for edge-facing APIs or when a real-time response is strictly required.
- Avoid chaining synchronous calls across service boundaries.

## Event Contracts

- Treat event schemas as public contracts.
- Version events explicitly using a `Version` property or namespace.
- Avoid breaking changes; prefer additive evolution.

```csharp
public sealed record OrderCreatedEvent
{
    public required Guid OrderId { get; init; }
    public required Guid CustomerId { get; init; }
    public required DateTimeOffset OccurredAt { get; init; }
    public required int Version { get; init; } = 1;
}
```

## Event Types

- Prefer domain events for business-significant occurrences.
- Use integration events for cross-service communication.
- Keep domain events internal to the service; translate to integration events at the boundary.

## Idempotency

- All event handlers must be idempotent.
- Use natural keys or idempotency keys to detect duplicate processing.

```csharp
public sealed class OrderCreatedHandler : IEventHandler<OrderCreatedEvent>
{
    public async Task HandleAsync(OrderCreatedEvent @event, CancellationToken ct)
    {
        if (await _orderRepository.ExistsAsync(@event.OrderId, ct))
        {
            return; // Already processed
        }

        // Process event...
    }
}
```

---

# Service-Oriented Architecture

## Service Boundaries

- Align service boundaries with business capabilities (bounded contexts).
- Each service owns its data exclusively; no shared databases between services.
- Avoid distributed transactions; use eventual consistency and sagas.

## Service Contracts

- Expose minimal surface area.
- Use stable, coarse-grained contracts.
- Avoid leaking internal domain models in integration events or APIs.

## Dependency Direction

- Services depend on abstractions, not on other service implementations.
- Prefer a thin anti-corruption layer at integration points.

```csharp
public interface IInventoryClient
{
    Task<Result> ReserveStockAsync(ReserveStockRequest request, CancellationToken ct);
}
```

---

# Domain-Driven Design

## Business Logic

Business rules belong in domain objects.

Prefer:

```csharp
user.Activate();
order.Cancel();
```

Avoid:

```csharp
user.IsActive = true;
order.Status = OrderStatus.Cancelled;
```

when domain behaviour exists.

---

## Entities

Entities should protect invariants.

Avoid exposing mutable state unnecessarily.

---

## Domain Events

Use domain events for side effects and cross-cutting concerns within a service.

Avoid coupling domain logic directly to infrastructure concerns.

### Publishing

- Raise domain events from aggregates.
- Dispatch domain events before or after persistence (depending on consistency requirements).
- Translate domain events to integration events at the application boundary; do not expose domain events directly outside the service.

---

# Resilience and Reliability

## Retry Policies

- Apply transient fault handling at infrastructure boundaries.
- Use jittered exponential backoff for retries.
- Avoid infinite retries; define a dead-letter mechanism.

## Circuit Breakers

- Use circuit breakers for synchronous HTTP calls to downstream services.
- Log state transitions (open, half-open, closed) for observability.

## Dead-Letter Queues

- Failed messages after retries must move to a dead-letter queue.
- Alert on dead-letter queue growth; do not silently drop failed events.

## Timeouts

- Always specify timeouts for external calls.
- Default to fail-fast rather than hang indefinitely.

---

# ASP.NET Core Standards

## API Style

Minimal APIs are preferred, but equally consider using controllers as per below.

Prefer:

```csharp
app.MapPost("/users", CreateUserHandler.HandleAsync);
```

```csharp
    [Route($"{Routes.BaseRoute.Name}")]
    public sealed class TestEndpoint : EndpointBase
    {
        [HttpGet]
        public override async Task<ActionResult> HandleAsync(CancellationToken cancellationToken = default)
        {
            return Ok();
        }
    }
```

---

## Endpoint Design

Endpoints should remain thin.

Responsibilities:

- Endpoint: HTTP concerns.
- Application: orchestration.
- Domain: business rules.
- Persistence: data access.

Avoid placing business logic directly inside endpoints.

---

## Dependency Injection

Use constructor injection exclusively.

Avoid:

- Service locators
- Static service providers
- Global state

Group registrations using extension methods.

```csharp
services.AddApplication();
services.AddPersistence();
services.AddInfrastructure();
```

---

## Configuration

Use strongly typed options.

Prefer:

```csharp
services.Configure<JwtOptions>(
    configuration.GetSection("Jwt"));
```

Avoid magic strings throughout the application.

---

## Logging

Use structured logging.

Prefer:

```csharp
_logger.LogInformation(
    "User {UserId} created",
    user.Id);
```

Avoid string interpolation in log messages.

Prefer using .NET Aspire Dashboard Standalone or .NET Aspire.

Always use extension methods that use the [LoggerMessage] attribute.

```csharp
public static partial class LoggingExtensions
{
    // ==========================================
    // Authentication Events (1000-1099)
    // ==========================================

    /// <summary>
    /// Logs successful user login.
    /// </summary>
    [LoggerMessage(
        EventId = 1001,
        Level = LogLevel.Information,
        Message = "User {UserId} logged in successfully from {IpAddress}")]
    public static partial void UserLoggedIn(
        this ILogger logger,
        string userId,
        string ipAddress);
}

```




## Distributed Tracing

- Propagate correlation IDs (`traceparent`, `correlation-id`) across all events and HTTP calls.
- Include correlation IDs in every log entry and event metadata.

```csharp
public sealed record OrderCreatedEvent
{
    public required Guid OrderId { get; init; }
    public required string CorrelationId { get; init; }
}
```

---

# Persistence Standards

## Entity Framework Core

Entity Framework Core is the default ORM.

Prefer Fluent API configuration.

Keep entity configuration separate from entities.

Prefer using compiled queries

Prefer:

```text
Persistence/
├── Configurations/
├── Interceptors/
├── Migrations/
└── Contexts/
└── Factories/
└── Seed/
```

Avoid excessive data annotations.

---

# Testing Standards

## Frameworks

Preferred stack:

- NUnit
- FluentAssertions
- Moq
- Testcontainers

---

## Test Naming

Use:

```text
Method_Should_ExpectedOutcome_When_Condition
```

Example:

```text
CreateUser_Should_ReturnUser_When_RequestIsValid
```

---

## Test Philosophy

Test behaviour rather than implementation details.

Prefer testing public behaviour over private implementation.

## Integration Testing

- Use Testcontainers for infrastructure dependencies (databases, message brokers).
- Test event publishing and consumption end-to-end within a service boundary.
- Avoid testing other services directly; use contract tests or mocks at boundaries.

---

# Security Standards

- Validate all external input.
- Use parameterized database queries.
- Never log secrets.
- Never store secrets in source control.
- Prefer least-privilege access.
- Use framework-provided authentication and authorization features.

## Inter-Service Security

- Authenticate and authorize service-to-service communication.
- Prefer mTLS or signed JWTs for service identities.
- Never trust events from external services without validation.
- Sanitize all data from integration events before processing.

---

# AI Agent Instructions

When generating code:

1. Preserve existing architecture.
2. Prefer modifying existing code over introducing abstractions.
3. Do not create base classes without demonstrated need.
4. Do not introduce generic repositories by default.
5. Do not introduce MediatR unless explicitly requested.
6. Do not introduce new NuGet packages without justification.
7. Prefer explicit implementations over framework-heavy solutions.
8. Keep methods small and focused.
9. Keep classes cohesive.
10. Add tests for behavioural changes.
11. Prefer readability over terseness.
12. If multiple solutions are valid, choose the simplest.
