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

## Part III: SonarQube C# Rules

- [SonarQube Overview](#sonarqube-overview)
- [Bugs](#bugs)
- [Vulnerabilities](#vulnerabilities)
- [Code Smells](#code-smells)
- [Security Hotspots](#security-hotspots)

## Part IV: AI Agent Instructions

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

# SonarQube Overview

All code must pass SonarQube analysis. The following sections catalogue the SonarQube C# rules organized by type: **Bugs**, **Vulnerabilities**, **Code Smells**, and **Security Hotspots**. Each rule includes a severity, a description, a noncompliant example, and the preferred compliant solution. Agents must ensure generated code adheres to these rules.

---

# Bugs

Rules that detect coding mistakes resulting in unexpected behaviour or crashes.

## S2583: Conditionally executed code should be reachable

**Severity:** Blocker  
**Description:** Code inside a conditional block that will never be executed indicates a logical error. The condition is always `true` or `false` given the current state, making the branch unreachable.

**Noncompliant:**

```csharp
public int Compute(int value)
{
    var result = value * 2;

    if (result > 0)
    {
        return result;
    }

    // This will never execute when value is positive
    if (result == 0)
    {
        return -1;
    }

    return 0;
}
```

**Compliant:**

```csharp
public int Compute(int value)
{
    var result = value * 2;

    if (result > 0)
    {
        return result;
    }

    if (result == 0)
    {
        return -1;
    }

    return 0;
}
```

---

## S2589: Boolean expressions should not be gratuitous

**Severity:** Major  
**Description:** Boolean expressions that are always `true` or always `false` are dead code and usually indicate a logical error.

**Noncompliant:**

```csharp
public void Process(string value)
{
    if (value is not null && value is not null)
    {
        // Redundant null check
    }
}
```

**Compliant:**

```csharp
public void Process(string value)
{
    if (value is not null)
    {
        // Single null check is sufficient
    }
}
```

---

## S1764: Identical expressions should not be used on both sides of a binary operator

**Severity:** Critical  
**Description:** Using the same variable on both sides of an operator (`a == a`, `a - a`) is either a mistake or pointless and should be removed.

**Noncompliant:**

```csharp
public int Calculate(int a, int b)
{
    return a - a; // Always 0
}
```

**Compliant:**

```csharp
public int Calculate(int a, int b)
{
    return a - b;
}
```

---

## S1145: Useless `if(true)` and `if(false)` checks should be removed

**Severity:** Major  
**Description:** Conditions that are hard-coded to `true` or `false` are dead code and should be simplified.

**Noncompliant:**

```csharp
public void DoWork()
{
    if (true)
    {
        Console.WriteLine("Always prints");
    }
}
```

**Compliant:**

```csharp
public void DoWork()
{
    Console.WriteLine("Always prints");
}
```

---

## S2184: Casting should not be performed on operands of a division before the division is performed

**Severity:** Major  
**Description:** Casting an operand to a narrower type before division can cause truncation and incorrect results. Cast the result of the division instead.

**Noncompliant:**

```csharp
public double GetAverage(int total, int count)
{
    return (double)total / count; // Fine
    return (double)(total / count); // Bug: integer division happens first
}
```

**Compliant:**

```csharp
public double GetAverage(int total, int count)
{
    return (double)total / count;
}
```

---

## S2187: TestCases should be defined for generic test classes

**Severity:** Critical  
**Description:** Generic test classes without concrete `TestCase` type arguments cannot be instantiated by test runners.

**Noncompliant:**

```csharp
public class GenericTests<T> { }
```

**Compliant:**

```csharp
[TestFixture(typeof(string))]
public class StringTests<T> { }
```

---

## S2197: Modulus results should not be checked for direct equality

**Severity:** Major  
**Description:** Checking `x % y == z` when `z` is not `0` or `y-1` is suspicious because modulus results are bounded to `[0, y-1]`.

**Noncompliant:**

```csharp
public bool IsValid(int value)
{
    return value % 2 == 3; // Always false
}
```

**Compliant:**

```csharp
public bool IsValid(int value)
{
    return value % 2 == 1;
}
```

---

## S2201: Return values from `System.Linq.Enumerable` should not be ignored

**Severity:** Major  
**Description:** LINQ methods return a new sequence and do not modify the source. Ignoring the return value means the operation had no effect.

**Noncompliant:**

```csharp
public void Filter(List<int> items)
{
    items.Where(x => x > 0); // Result is discarded
}
```

**Compliant:**

```csharp
public List<int> Filter(List<int> items)
{
    return items.Where(x => x > 0).ToList();
}
```

---

## S2259: Null pointers should not be dereferenced

**Severity:** Blocker  
**Description:** Dereferencing a variable that is known to be `null` will cause a `NullReferenceException`.

**Noncompliant:**

```csharp
public int GetLength(string value)
{
    if (value is null)
    {
        return 0;
    }

    return value.Length; // If null check is removed elsewhere, this crashes
}
```

**Compliant:**

```csharp
public int GetLength(string value)
{
    return value?.Length ?? 0;
}
```

---

## S2757: `=+` should not be used instead of `+=`

**Severity:** Critical  
**Description:** Writing `a =+ b` assigns the unary-positive of `b` to `a` instead of adding `b` to `a`.

**Noncompliant:**

```csharp
public void Accumulate(int total, int value)
{
    total =+ value; // Bug: equivalent to total = +value
}
```

**Compliant:**

```csharp
public void Accumulate(int total, int value)
{
    total += value;
}
```

---

## S3168: `async` methods should not return `void`

**Severity:** Critical  
**Description:** `async void` methods cannot be awaited and exceptions thrown inside them crash the process.

**Noncompliant:**

```csharp
public async void ProcessAsync()
{
    await Task.Delay(100);
}
```

**Compliant:**

```csharp
public async Task ProcessAsync()
{
    await Task.Delay(100);
}
```

---

## S3172: `GetHashCode` should not reference mutable fields

**Severity:** Critical  
**Description:** If a field used in `GetHashCode` changes while the object is stored in a hash-based collection, the object becomes unreachable.

**Noncompliant:**

```csharp
public class User
{
    public string Name { get; set; }

    public override int GetHashCode()
    {
        return Name.GetHashCode();
    }
}
```

**Compliant:**

```csharp
public sealed class User
{
    public required string Name { get; init; }

    public override int GetHashCode()
    {
        return Name.GetHashCode();
    }
}
```

---

## S3215: `GetType()` should not be called on `System.Type` instances

**Severity:** Major  
**Description:** Calling `GetType()` on a `System.Type` instance returns `typeof(Type)` rather than the represented type.

**Noncompliant:**

```csharp
public bool IsString(Type type)
{
    return type.GetType() == typeof(string); // Always false
}
```

**Compliant:**

```csharp
public bool IsString(Type type)
{
    return type == typeof(string);
}
```

---

## S3244: Anonymous delegates should not be used to unsubscribe from events

**Severity:** Major  
**Description:** Unsubscribing an anonymous delegate that is not the exact same instance does nothing.

**Noncompliant:**

```csharp
eventHandler -= (s, e) => { };
```

**Compliant:**

```csharp
EventHandler handler = (s, e) => { };
eventHandler += handler;
eventHandler -= handler;
```

---

## S3443: `Format` strings should be used correctly

**Severity:** Major  
**Description:** Passing a composite format string as an argument to another `string.Format` can cause incorrect placeholder indexing.

**Noncompliant:**

```csharp
public string BuildMessage(string name)
{
    return string.Format("Name: {0}", string.Format("User: {0}", name));
}
```

**Compliant:**

```csharp
public string BuildMessage(string name)
{
    return $"Name: User: {name}";
}
```

---

## S3464: `Assembly.GetCallingAssembly()` should not be used in `async` methods

**Severity:** Major  
**Description:** In `async` methods, the calling assembly is the framework rather than the expected caller.

**Noncompliant:**

```csharp
public async Task DoWorkAsync()
{
    var caller = Assembly.GetCallingAssembly();
}
```

**Compliant:**

```csharp
public void DoWork()
{
    var caller = Assembly.GetCallingAssembly();
}
```

---

## S3655: `Nullable` value should be accessed only when it is known to be non-null

**Severity:** Blocker  
**Description:** Accessing `Value` on a nullable when it is known to be null throws `InvalidOperationException`.

**Noncompliant:**

```csharp
public int GetValue(int? value)
{
    if (value is null)
    {
        return value.Value; // Throws
    }

    return 0;
}
```

**Compliant:**

```csharp
public int GetValue(int? value)
{
    return value ?? 0;
}
```

---

## S3880: `NetDataContractSerializer` should not be used

**Severity:** Blocker  
**Description:** This serializer is insecure and can execute arbitrary code during deserialization.

**Noncompliant:**

```csharp
var serializer = new NetDataContractSerializer();
```

**Compliant:**

```csharp
var serializer = new DataContractSerializer(typeof(MyType));
```

---

## S3984: `Exception` should not be created without being thrown

**Severity:** Major  
**Description:** Creating an exception but not throwing it is almost always a mistake.

**Noncompliant:**

```csharp
public void Validate(string value)
{
    if (value is null)
    {
        new ArgumentNullException(nameof(value));
    }
}
```

**Compliant:**

```csharp
public void Validate(string value)
{
    ArgumentNullException.ThrowIfNull(value);
}
```

---

## S4143: Collection elements should not be replaced unconditionally

**Severity:** Major  
**Description:** Adding an item to a dictionary with the same key twice means the second write overwrites the first unconditionally.

**Noncompliant:**

```csharp
var map = new Dictionary<string, int>();
map["key"] = 1;
map["key"] = 2; // First assignment is lost
```

**Compliant:**

```csharp
var map = new Dictionary<string, int>();
map["key"] = 2;
```

---

## S4462: `Task.Wait` should not be called in an `async` method

**Severity:** Major  
**Description:** Calling `Task.Wait` or `Task.Result` in an async method causes a deadlock in many contexts and defeats the purpose of `async`/`await`.

**Noncompliant:**

```csharp
public async Task ProcessAsync()
{
    var task = DoWorkAsync();
    task.Wait(); // Deadlock risk
}
```

**Compliant:**

```csharp
public async Task ProcessAsync()
{
    await DoWorkAsync();
}
```

---

## S4581: `new Guid()` should not be used

**Severity:** Critical  
**Description:** `new Guid()` creates an all-zero GUID, which is almost never the intended behaviour.

**Noncompliant:**

```csharp
var id = new Guid();
```

**Compliant:**

```csharp
var id = Guid.NewGuid();
```

---

## S6612: The value of `bool` should not be negated more than once

**Severity:** Major  
**Description:** Multiple negations of a boolean are confusing and error-prone.

**Noncompliant:**

```csharp
public bool IsNotInactive(bool active)
{
    return !!!active;
}
```

**Compliant:**

```csharp
public bool IsNotInactive(bool active)
{
    return !active;
}
```

---

## S6613: Casting to `float` should not be done before calling `Math.Floor`

**Severity:** Major  
**Description:** Casting to `float` before `Math.Floor` loses precision. Use `Math.Floor(double)` or `MathF.Floor(float)`.

**Noncompliant:**

```csharp
public int Round(float value)
{
    return (int)Math.Floor((double)value);
}
```

**Compliant:**

```csharp
public int Round(float value)
{
    return (int)MathF.Floor(value);
}
```

---

# Vulnerabilities

Rules that detect security weaknesses that can be exploited by attackers.

## S3329: Cipher Block Chaining IVs should be unpredictable

**Severity:** Blocker  
**Description:** Using a predictable or hardcoded IV in CBC mode encryption makes the ciphertext vulnerable to attacks.

**Noncompliant:**

```csharp
public byte[] Encrypt(byte[] data, byte[] key)
{
    var iv = new byte[16]; // All zeros
    using var aes = Aes.Create();
    aes.IV = iv;
    aes.Key = key;
    // ...
}
```

**Compliant:**

```csharp
public byte[] Encrypt(byte[] data, byte[] key)
{
    using var aes = Aes.Create();
    aes.Key = key;
    aes.GenerateIV();
    // ...
}
```

---

## S3330: Server-side requests should not be vulnerable to SSRF attacks

**Severity:** Blocker  
**Description:** Server-Side Request Forgery occurs when user input is used to construct a request to an internal or external resource without validation.

**Noncompliant:**

```csharp
public async Task<string> FetchData(string url)
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);
}
```

**Compliant:**

```csharp
public async Task<string> FetchData(string url)
{
    if (!Uri.TryCreate(url, UriKind.Absolute, out var uri)
        || uri.Host != "allowed.example.com")
    {
        throw new ArgumentException("Invalid URL");
    }

    using var client = new HttpClient();
    return await client.GetStringAsync(uri);
}
```

---

## S4423: Weak SSL/TLS protocols should not be used

**Severity:** Blocker  
**Description:** Using SSL 3.0, TLS 1.0, or TLS 1.1 exposes the application to known cryptographic attacks.

**Noncompliant:**

```csharp
var handler = new HttpClientHandler
{
    SslProtocols = SslProtocols.Tls
};
```

**Compliant:**

```csharp
var handler = new HttpClientHandler
{
    SslProtocols = SslProtocols.Tls13
};
```

---

## S4426: Cryptographic keys should not be too short

**Severity:** Blocker  
**Description:** Short keys are vulnerable to brute-force attacks. Use at least 2048-bit RSA or 256-bit elliptic curve keys.

**Noncompliant:**

```csharp
using var rsa = RSA.Create(1024);
```

**Compliant:**

```csharp
using var rsa = RSA.Create(2048);
```

---

## S4428: TempFileCollection should not be used

**Severity:** Major  
**Description:** `TempFileCollection` can create files with weak permissions. Use `Path.GetTempFileName` with explicit cleanup.

**Noncompliant:**

```csharp
var tempFiles = new TempFileCollection();
```

**Compliant:**

```csharp
var tempPath = Path.GetTempFileName();
try
{
    // Use file
}
finally
{
    File.Delete(tempPath);
}
```

---

## S4784: Using regular expressions is security-sensitive

**Severity:** Critical  
**Description:** Regular expressions can be vulnerable to ReDoS (Regular Expression Denial of Service) if user input is used without validation.

**Noncompliant:**

```csharp
public bool IsValidEmail(string email)
{
    return Regex.IsMatch(email, @"^.*@.*$");
}
```

**Compliant:**

```csharp
public bool IsValidEmail(string email)
{
    if (email.Length > 254)
    {
        return false;
    }

    return Regex.IsMatch(email, @"^[^@]+@[^@]+$", RegexOptions.None, TimeSpan.FromMilliseconds(100));
}
```

---

## S4787: Cryptographic keys should not be hardcoded

**Severity:** Blocker  
**Description:** Hardcoded keys are exposed in source control and binaries. Use key management services or configuration.

**Noncompliant:**

```csharp
private static readonly byte[] Key = Encoding.UTF8.GetBytes("hardcoded-key-1234");
```

**Compliant:**

```csharp
private readonly byte[] _key;

public CryptoService(IOptions<CryptoOptions> options)
{
    _key = Convert.FromBase64String(options.Value.Key);
}
```

---

## S4790: Hashing data is security-sensitive

**Severity:** Critical  
**Description:** Weak hash algorithms like MD5 or SHA-1 are vulnerable to collision attacks. Use SHA-256 or stronger.

**Noncompliant:**

```csharp
public byte[] Hash(string input)
{
    using var md5 = MD5.Create();
    return md5.ComputeHash(Encoding.UTF8.GetBytes(input));
}
```

**Compliant:**

```csharp
public byte[] Hash(string input)
{
    using var sha256 = SHA256.Create();
    return sha256.ComputeHash(Encoding.UTF8.GetBytes(input));
}
```

---

## S4818: Sockets should not be used in production code without proper validation

**Severity:** Major  
**Description:** Raw socket access can bypass security controls. Validate all endpoints and use TLS.

**Noncompliant:**

```csharp
var client = new TcpClient();
await client.ConnectAsync("untrusted.com", 80);
```

**Compliant:**

```csharp
// Use HttpClient with validated URLs over TLS
```

---

## S5324: `Random` should not be used for security-sensitive purposes

**Severity:** Blocker  
**Description:** `System.Random` is predictable and should never be used for cryptography, tokens, or passwords.

**Noncompliant:**

```csharp
public string GenerateToken()
{
    var random = new Random();
    return random.Next().ToString();
}
```

**Compliant:**

```csharp
public string GenerateToken()
{
    var bytes = new byte[32];
    RandomNumberGenerator.Fill(bytes);
    return Convert.ToHexString(bytes);
}
```

---

## S5542: Encryption algorithms should be used with secure mode and padding scheme

**Severity:** Blocker  
**Description:** Using ECB mode or no padding exposes the ciphertext to attacks. Use CBC or GCM with proper IVs.

**Noncompliant:**

```csharp
using var aes = Aes.Create();
aes.Mode = CipherMode.ECB;
```

**Compliant:**

```csharp
using var aes = Aes.Create();
aes.Mode = CipherMode.CBC;
aes.GenerateIV();
```

---

## S5547: Cipher algorithms should be robust

**Severity:** Blocker  
**Description:** Weak or broken algorithms like DES, RC2, or TripleDES should not be used.

**Noncompliant:**

```csharp
using var des = DES.Create();
```

**Compliant:**

```csharp
using var aes = Aes.Create();
```

---

## S5659: SSL/TLS connection settings should not be bypassed

**Severity:** Blocker  
**Description:** Disabling certificate validation exposes the application to man-in-the-middle attacks.

**Noncompliant:**

```csharp
ServicePointManager.ServerCertificateValidationCallback = (sender, cert, chain, errors) => true;
```

**Compliant:**

```csharp
// Use default certificate validation
```

---

## S5773: Types allowed to be deserialized should be limited

**Severity:** Blocker  
**Description:** Deserializing arbitrary types can lead to remote code execution. Whitelist allowed types.

**Noncompliant:**

```csharp
var formatter = new BinaryFormatter();
var obj = formatter.Deserialize(stream);
```

**Compliant:**

```csharp
// Use JSON deserialization with type constraints
// or BinaryFormatter with a SerializationBinder whitelist
```

---

## S6419: Cryptographic keys should not be too short for the algorithm

**Severity:** Blocker  
**Description:** Using AES-128 is acceptable, but AES-256 is preferred. Using shorter keys than the algorithm supports causes vulnerabilities.

**Noncompliant:**

```csharp
using var aes = Aes.Create();
aes.KeySize = 64;
```

**Compliant:**

```csharp
using var aes = Aes.Create();
aes.KeySize = 256;
```

---

---

# Code Smells

Rules that detect maintainability issues, suspicious coding practices, and patterns that may lead to bugs or make code harder to understand and maintain.

## S1075: URIs should not be hardcoded

**Severity:** Minor  
**Description:** Hardcoded URIs make the application inflexible and can expose internal endpoints in source control.

**Noncompliant:**

```csharp
public string GetApiUrl()
{
    return "https://api.example.com/v1/users";
}
```

**Compliant:**

```csharp
public string GetApiUrl(IOptions<ApiOptions> options)
{
    return options.Value.BaseUrl;
}
```

---

## S1116: Empty statements should be removed

**Severity:** Minor  
**Description:** A stray semicolon creates an empty statement that can hide bugs and confuse readers.

**Noncompliant:**

```csharp
public void DoWork();
{
    Console.WriteLine("work");
}
```

**Compliant:**

```csharp
public void DoWork()
{
    Console.WriteLine("work");
}
```

---

## S1121: Assignments should not be made from within sub-expressions

**Severity:** Major  
**Description:** Embedding an assignment inside another expression makes the code harder to read and can hide bugs.

**Noncompliant:**

```csharp
public bool TryParse(string input)
{
    int value;
    return int.TryParse(input, out value);
}
```

**Compliant:**

```csharp
public bool TryParse(string input, out int value)
{
    return int.TryParse(input, out value);
}
```

---

## S1125: Boolean literals should not be redundant

**Severity:** Minor  
**Description:** Expressions like `x == true` or `y == false` are redundant and should be simplified.

**Noncompliant:**

```csharp
public bool IsValid(bool flag)
{
    return flag == true;
}
```

**Compliant:**

```csharp
public bool IsValid(bool flag)
{
    return flag;
}
```

---

## S1134: Track uses of `TODO` tags

**Severity:** Info  
**Description:** `TODO` comments indicate incomplete work. They should be tracked and resolved before merging.

**Noncompliant:**

```csharp
// TODO: implement validation
public void Process() { }
```

**Compliant:**

```csharp
public void Process()
{
    // Validation implemented
}
```

---

## S1135: Track uses of `FIXME` tags

**Severity:** Info  
**Description:** `FIXME` comments indicate known defects. They should be tracked and resolved before merging.

**Noncompliant:**

```csharp
// FIXME: this breaks under load
public void Process() { }
```

**Compliant:**

```csharp
public void Process()
{
    // Fixed and load-tested
}
```

---

## S1147: Exit methods should not be called

**Severity:** Major  
**Description:** Calling `Environment.Exit` or `Process.Kill` inside a library or web application abruptly terminates the process and loses data.

**Noncompliant:**

```csharp
public void Shutdown()
{
    Environment.Exit(1);
}
```

**Compliant:**

```csharp
public void Shutdown(ILifetime lifetime)
{
    lifetime.StopApplication();
}
```

---

## S1155: `Any()` should be used to test for emptiness

**Severity:** Minor  
**Description:** `Count() > 0` or `Count() == 0` is less efficient and less readable than `Any()` or `!Any()`.

**Noncompliant:**

```csharp
public bool HasItems(IEnumerable<int> items)
{
    return items.Count() > 0;
}
```

**Compliant:**

```csharp
public bool HasItems(IEnumerable<int> items)
{
    return items.Any();
}
```

---

## S1168: Empty arrays and collections should be returned instead of `null`

**Severity:** Major  
**Description:** Returning `null` for a collection forces every caller to add a null check. Return an empty collection instead.

**Noncompliant:**

```csharp
public List<int> GetItems()
{
    return null;
}
```

**Compliant:**

```csharp
public List<int> GetItems()
{
    return [];
}
```

---

## S1172: Unused method parameters should be removed

**Severity:** Minor  
**Description:** Parameters that are never used add noise to the API and confuse callers.

**Noncompliant:**

```csharp
public int Add(int a, int b, int unused)
{
    return a + b;
}
```

**Compliant:**

```csharp
public int Add(int a, int b)
{
    return a + b;
}
```

---

## S1186: Methods should not be empty

**Severity:** Major  
**Description:** Empty methods serve no purpose and should either be implemented or removed.

**Noncompliant:**

```csharp
public void Validate()
{
}
```

**Compliant:**

```csharp
public void Validate()
{
    ArgumentNullException.ThrowIfNull(_value);
}
```

---

## S1199: Nested code blocks should not be used

**Severity:** Minor  
**Description:** Unnecessary nested blocks create unnecessary scope and confuse readers.

**Noncompliant:**

```csharp
public void Process()
{
    {
        var value = 1;
    }
}
```

**Compliant:**

```csharp
public void Process()
{
    var value = 1;
}
```

---

## S1226: Method parameters should not be reassigned

**Severity:** Major  
**Description:** Reassigning a parameter inside a method makes the code harder to follow.

**Noncompliant:**

```csharp
public int Compute(int value)
{
    value = value * 2;
    return value;
}
```

**Compliant:**

```csharp
public int Compute(int value)
{
    var result = value * 2;
    return result;
}
```

---

## S125: Sections of code should not be commented out

**Severity:** Major  
**Description:** Commented-out code clutters the file and is not maintained. Remove it or use version control.

**Noncompliant:**

```csharp
// public void OldMethod()
// {
//     Console.WriteLine("old");
// }
```

**Compliant:**

```csharp
// Code removed; see git history if needed.
```

---

## S131: `switch` statements should have a `default` clause

**Severity:** Major  
**Description:** A `switch` without `default` may silently miss cases when new enum values are added.

**Noncompliant:**

```csharp
public string GetLabel(Status status)
{
    switch (status)
    {
        case Status.Active:
            return "Active";
        case Status.Inactive:
            return "Inactive";
    }

    return string.Empty;
}
```

**Compliant:**

```csharp
public string GetLabel(Status status)
{
    return status switch
    {
        Status.Active => "Active",
        Status.Inactive => "Inactive",
        _ => throw new ArgumentOutOfRangeException(nameof(status))
    };
}
```

---

## S1481: Unused local variables should be removed

**Severity:** Minor  
**Description:** Declared variables that are never used create noise and may indicate incomplete logic.

**Noncompliant:**

```csharp
public void Process()
{
    var value = 1;
    Console.WriteLine("done");
}
```

**Compliant:**

```csharp
public void Process()
{
    Console.WriteLine("done");
}
```

---

## S1643: Strings should not be concatenated using `+` in a loop

**Severity:** Minor  
**Description:** Repeated string concatenation in a loop creates many intermediate string objects. Use `StringBuilder`.

**Noncompliant:**

```csharp
public string BuildString(List<string> parts)
{
    var result = "";
    foreach (var part in parts)
    {
        result += part;
    }

    return result;
}
```

**Compliant:**

```csharp
public string BuildString(List<string> parts)
{
    var builder = new StringBuilder();
    foreach (var part in parts)
    {
        builder.Append(part);
    }

    return builder.ToString();
}
```

---

## S1854: Unused assignments should be removed

**Severity:** Major  
**Description:** Assigning a value to a variable and then overwriting it before reading is dead code.

**Noncompliant:**

```csharp
public int Compute()
{
    var value = 1;
    value = 2;
    return value;
}
```

**Compliant:**

```csharp
public int Compute()
{
    var value = 2;
    return value;
}
```

---

## S1905: Redundant casts should not be used

**Severity:** Minor  
**Description:** Casting a value to its own type or to a base type it already is does nothing.

**Noncompliant:**

```csharp
public string GetValue(object value)
{
    return (string)value.ToString();
}
```

**Compliant:**

```csharp
public string GetValue(object value)
{
    return value.ToString();
}
```

---

## S1939: Inheritance list should not be redundant

**Severity:** Minor  
**Description:** Explicitly listing an interface that is already inherited through a base class is redundant.

**Noncompliant:**

```csharp
public class MyDisposable : IDisposable, IAsyncDisposable
{
    // IAsyncDisposable already inherits IDisposable
}
```

**Compliant:**

```csharp
public class MyDisposable : IAsyncDisposable
{
}
```

---

## S1940: Aborting `finally` blocks should not be used

**Severity:** Blocker  
**Description:** Throwing an exception or calling `Environment.Exit` in a `finally` block suppresses the original exception and loses diagnostic information.

**Noncompliant:**

```csharp
public void Process()
{
    try
    {
        DoWork();
    }
    finally
    {
        throw new InvalidOperationException("Cleanup failed");
    }
}
```

**Compliant:**

```csharp
public void Process()
{
    try
    {
        DoWork();
    }
    finally
    {
        Cleanup();
    }
}
```

---

## S1994: `GC.SuppressFinalize` should not be called on `IDisposable` types that do not have a finalizer

**Severity:** Major  
**Description:** Calling `GC.SuppressFinalize(this)` in a `Dispose` method when the class has no finalizer is unnecessary.

**Noncompliant:**

```csharp
public class Resource : IDisposable
{
    public void Dispose()
    {
        GC.SuppressFinalize(this);
    }
}
```

**Compliant:**

```csharp
public class Resource : IDisposable
{
    public void Dispose()
    {
        // No finalizer, so no SuppressFinalize needed
    }
}
```

---

## S2114: `Equality` checks should not be made with `float.NaN`

**Severity:** Major  
**Description:** `float.NaN == float.NaN` is always `false`. Use `float.IsNaN` instead.

**Noncompliant:**

```csharp
public bool IsNotANumber(float value)
{
    return value == float.NaN;
}
```

**Compliant:**

```csharp
public bool IsNotANumber(float value)
{
    return float.IsNaN(value);
}
```

---

## S2208: `System.Console` should not be used in production code

**Severity:** Minor  
**Description:** Writing to the console from production code pollutes logs and is not suitable for structured logging.

**Noncompliant:**

```csharp
public void Process()
{
    Console.WriteLine("Processing started");
}
```

**Compliant:**

```csharp
public void Process(ILogger<Processor> logger)
{
    logger.LogInformation("Processing started");
}
```

---

## S2223: Non-constant static fields should not be visible

**Severity:** Major  
**Description:** Mutable static fields are shared across all threads and can cause race conditions.

**Noncompliant:**

```csharp
public class Config
{
    public static string ConnectionString;
}
```

**Compliant:**

```csharp
public static class Config
{
    public static string ConnectionString { get; set; }
}
```

---

## S2225: `ToString()` should not return `null`

**Severity:** Major  
**Description:** Returning `null` from `ToString()` violates the contract and causes `NullReferenceException` in callers.

**Noncompliant:**

```csharp
public override string ToString()
{
    return null;
}
```

**Compliant:**

```csharp
public override string ToString()
{
    return string.Empty;
}
```

---

## S2234: Parameters should be passed in the correct order

**Severity:** Major  
**Description:** Passing arguments in the wrong order can cause subtle bugs, especially when parameter types match.

**Noncompliant:**

```csharp
public void Move(int x, int y)
{
    // x and y swapped accidentally
    _cursor.Move(y, x);
}
```

**Compliant:**

```csharp
public void Move(int x, int y)
{
    _cursor.Move(x, y);
}
```

---

## S2325: Methods and properties that do not access instance data should be static

**Severity:** Minor  
**Description:** Methods that do not use instance members should be `static` to indicate they are stateless.

**Noncompliant:**

```csharp
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}
```

**Compliant:**

```csharp
public class Calculator
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}
```

---

## S2326: Unused type parameters should be removed

**Severity:** Major  
**Description:** Generic type parameters that are not used in the method signature serve no purpose.

**Noncompliant:**

```csharp
public TResult Process<TResult>(string input)
{
    return default;
}
```

**Compliant:**

```csharp
public string Process(string input)
{
    return input;
}
```

---

## S2357: Fields should not have public accessibility

**Severity:** Minor  
**Description:** Public fields break encapsulation and cannot have validation or change notification added later.

**Noncompliant:**

```csharp
public class User
{
    public string Name;
}
```

**Compliant:**

```csharp
public class User
{
    public required string Name { get; init; }
}
```

---

## S2436: Classes and methods should not have too many generic parameters

**Severity:** Major  
**Description:** More than three generic parameters make the API hard to use and indicate a design issue.

**Noncompliant:**

```csharp
public class Container<T1, T2, T3, T4>
{
}
```

**Compliant:**

```csharp
public class Container<TKey, TValue>
{
}
```

---

## S2486: Generic exceptions should not be thrown

**Severity:** Major  
**Description:** Throwing `Exception`, `SystemException`, or `ApplicationException` forces callers to catch all exceptions, hiding specific failure modes.

**Noncompliant:**

```csharp
public void Process()
{
    throw new Exception("Failed");
}
```

**Compliant:**

```csharp
public void Process()
{
    throw new InvalidOperationException("Failed");
}
```

---

## S2589: Boolean expressions should not be gratuitous

**Severity:** Major  
**Description:** Conditions that are always `true` or always `false` are dead code and usually indicate a logic error.

**Noncompliant:**

```csharp
public void Process(string value)
{
    if (value is not null && value is not null)
    {
        // Redundant null check
    }
}
```

**Compliant:**

```csharp
public void Process(string value)
{
    if (value is not null)
    {
        // Single null check is sufficient
    }
}
```

---

## S2699: Tests should include assertions

**Severity:** Blocker  
**Description:** Tests without assertions pass even when the code under test is broken.

**Noncompliant:**

```csharp
[Test]
public void CreateUser_ReturnsUser()
{
    var user = _service.CreateUser("test");
    // No assertion
}
```

**Compliant:**

```csharp
[Test]
public void CreateUser_ReturnsUser()
{
    var user = _service.CreateUser("test");
    user.Should().NotBeNull();
    user.Name.Should().Be("test");
}
```

---

## S2971: `IEnumerable` LINQ methods should be simplified

**Severity:** Minor  
**Description:** Calling `Count()` then `Any()` on the same sequence is redundant. Use the result directly.

**Noncompliant:**

```csharp
public bool HasItems(List<int> items)
{
    return items.Count() > 0 && items.Any();
}
```

**Compliant:**

```csharp
public bool HasItems(List<int> items)
{
    return items.Any();
}
```

---

## S3052: Members should not be initialized to default values

**Severity:** Minor  
**Description:** Explicitly initializing a field or property to its default value is redundant.

**Noncompliant:**

```csharp
public class User
{
    public bool IsActive { get; set; } = false;
    public int Age { get; set; } = 0;
}
```

**Compliant:**

```csharp
public class User
{
    public bool IsActive { get; set; }
    public int Age { get; set; }
}
```

---

## S3060: `is` should be used instead of `==` for type comparison

**Severity:** Major  
**Description:** Using `== typeof(...)` is less readable than the `is` operator and does not handle inheritance correctly.

**Noncompliant:**

```csharp
public bool IsString(object value)
{
    return value.GetType() == typeof(string);
}
```

**Compliant:**

```csharp
public bool IsString(object value)
{
    return value is string;
}
```

---

## S3063: Strings should not be concatenated with `+` when the result is immediately discarded

**Severity:** Major  
**Description:** Concatenating strings and not using the result is dead code and indicates a mistake.

**Noncompliant:**

```csharp
public void Build()
{
    var a = "hello";
    a + " world"; // Result discarded
}
```

**Compliant:**

```csharp
public string Build()
{
    var a = "hello";
    return a + " world";
}
```

---

## S3235: Redundant `Parentheses` should not be used

**Severity:** Minor  
**Description:** Extra parentheses around a simple expression add noise without improving readability.

**Noncompliant:**

```csharp
public int Compute(int a)
{
    return (a + 1);
}
```

**Compliant:**

```csharp
public int Compute(int a)
{
    return a + 1;
}
```

---

## S3240: The simplest possible condition syntax should be used

**Severity:** Minor  
**Description:** Prefer ternary expressions or `??` over `if/else` when both branches are simple assignments or returns.

**Noncompliant:**

```csharp
public string GetLabel(bool active)
{
    if (active)
    {
        return "Active";
    }
    else
    {
        return "Inactive";
    }
}
```

**Compliant:**

```csharp
public string GetLabel(bool active)
{
    return active ? "Active" : "Inactive";
}
```

---

## S3241: Methods should not return values that are never used

**Severity:** Major  
**Description:** Returning a value that callers ignore is misleading and suggests the return type should be `void`.

**Noncompliant:**

```csharp
public int Process()
{
    DoWork();
    return 0; // Caller never uses this
}
```

**Compliant:**

```csharp
public void Process()
{
    DoWork();
}
```

---

## S3253: Constructor initializers should not be redundant

**Severity:** Minor  
**Description:** Calling `: base()` explicitly is redundant because it is done implicitly.

**Noncompliant:**

```csharp
public class User : Entity
{
    public User() : base()
    {
    }
}
```

**Compliant:**

```csharp
public class User : Entity
{
    public User()
    {
    }
}
```

---

## S3254: Default parameter values should not be passed as arguments

**Severity:** Minor  
**Description:** Passing the same value that is already the parameter default is redundant.

**Noncompliant:**

```csharp
public void Process(int value = 10)
{
    Process(value: 10);
}
```

**Compliant:**

```csharp
public void Process(int value = 10)
{
    Process();
}
```

---

## S3261: Namespaces should not be empty

**Severity:** Minor  
**Description:** Empty namespaces serve no purpose and should be removed.

**Noncompliant:**

```csharp
namespace MyApplication.Empty
{
}
```

**Compliant:**

```csharp
namespace MyApplication.Features
{
    public class Feature { }
}
```

---

## S3262: `params` should be used on overrides

**Severity:** Minor  
**Description:** When overriding a method with `params`, the override should also use `params` to preserve caller flexibility.

**Noncompliant:**

```csharp
public class Base
{
    public virtual void Log(params string[] messages) { }
}

public class Derived : Base
{
    public override void Log(string[] messages) { }
}
```

**Compliant:**

```csharp
public class Base
{
    public virtual void Log(params string[] messages) { }
}

public class Derived : Base
{
    public override void Log(params string[] messages) { }
}
```

---

## S3263: Static fields should appear in the order they must be initialized

**Severity:** Major  
**Description:** If a static field initializer depends on another static field, the dependent field must be declared after the one it depends on.

**Noncompliant:**

```csharp
public class Config
{
    public static readonly string FullUrl = BaseUrl + "/api";
    public static readonly string BaseUrl = "https://example.com";
}
```

**Compliant:**

```csharp
public class Config
{
    public static readonly string BaseUrl = "https://example.com";
    public static readonly string FullUrl = BaseUrl + "/api";
}
```

---

## S3264: Event handlers should have the correct signature

**Severity:** Major  
**Description:** Event handlers should follow the `(object sender, EventArgs e)` pattern for compatibility.

**Noncompliant:**

```csharp
public void OnClick()
{
}
```

**Compliant:**

```csharp
public void OnClick(object sender, EventArgs e)
{
}
```

---

## S3265: Non-flags enums should not be marked with `FlagsAttribute`

**Severity:** Major  
**Description:** Applying `[Flags]` to an enum whose values are not powers of two causes incorrect bitwise behaviour.

**Noncompliant:**

```csharp
[Flags]
public enum Status
{
    Active = 1,
    Inactive = 2,
    Pending = 3
}
```

**Compliant:**

```csharp
public enum Status
{
    Active = 1,
    Inactive = 2,
    Pending = 3
}
```

---

## S3353: Uninitialized fields should not be used

**Severity:** Major  
**Description:** Reading a field that has not been initialized yields the default value and usually indicates a missing constructor assignment.

**Noncompliant:**

```csharp
public class Counter
{
    private int _count;

    public int GetCount()
    {
        return _count; // Uninitialized, returns 0
    }
}
```

**Compliant:**

```csharp
public class Counter
{
    private int _count;

    public Counter()
    {
        _count = 0;
    }

    public int GetCount()
    {
        return _count;
    }
}
```

---

## S3366: `this` should not be passed out of a constructor

**Severity:** Major  
**Description:** Passing `this` from a constructor allows other code to observe the object before it is fully constructed.

**Noncompliant:**

```csharp
public class User
{
    public User(Registry registry)
    {
        registry.Register(this);
    }
}
```

**Compliant:**

```csharp
public class User
{
    public void RegisterWith(Registry registry)
    {
        registry.Register(this);
    }
}
```

---

## S3431: `Assert.AreEqual` should not be used with `float` or `double`

**Severity:** Major  
**Description:** Exact equality on floating-point numbers is unreliable due to precision issues.

**Noncompliant:**

```csharp
Assert.AreEqual(0.1 + 0.2, 0.3);
```

**Compliant:**

```csharp
Assert.AreEqual(0.3, 0.1 + 0.2, 0.0001);
```

---

## S3600: `Assembly.GetExecutingAssembly` should not be called in `async` methods

**Severity:** Major  
**Description:** In `async` methods, the executing assembly is the framework rather than the expected caller.

**Noncompliant:**

```csharp
public async Task DoWorkAsync()
{
    var asm = Assembly.GetExecutingAssembly();
}
```

**Compliant:**

```csharp
public void DoWork()
{
    var asm = Assembly.GetExecutingAssembly();
}
```

---

## S3626: Jump statements should not be redundant

**Severity:** Minor  
**Description:** `return`, `break`, or `continue` at the end of a block where control would flow there anyway is redundant.

**Noncompliant:**

```csharp
public void Process()
{
    if (true)
    {
        DoWork();
        return;
    }
}
```

**Compliant:**

```csharp
public void Process()
{
    DoWork();
}
```

---

## S3693: Constructor arguments should not be passed to `base` with the same name and type

**Severity:** Minor  
**Description:** Passing a constructor parameter to `base` with the same name is unnecessary when the base constructor is called implicitly.

**Noncompliant:**

```csharp
public class Derived : Base
{
    public Derived(string name) : base(name)
    {
    }
}
```

**Compliant:**

```csharp
public class Derived : Base
{
    public Derived(string name)
    {
    }
}
```

---

## S3776: Cognitive Complexity of methods should not be too high

**Severity:** Critical  
**Description:** Methods with high cognitive complexity (default threshold 15) are hard to understand and maintain. Refactor into smaller methods.

**Noncompliant:**

```csharp
public void Process(Order order)
{
    if (order is not null)
    {
        if (order.Items is not null)
        {
            foreach (var item in order.Items)
            {
                if (item.Quantity > 0)
                {
                    if (item.Price > 0)
                    {
                        // ... deeply nested logic
                    }
                }
            }
        }
    }
}
```

**Compliant:**

```csharp
public void Process(Order order)
{
    if (order is null || order.Items is null)
    {
        return;
    }

    foreach (var item in order.Items)
    {
        ProcessItem(item);
    }
}

private void ProcessItem(Item item)
{
    if (item.Quantity <= 0 || item.Price <= 0)
    {
        return;
    }

    // ...
}
```

---

## S3869: `SafeHandle` should be used instead of `IntPtr`

**Severity:** Major  
**Description:** Using `IntPtr` for handles does not provide automatic resource management. Use `SafeHandle` derivatives.

**Noncompliant:**

```csharp
[DllImport("kernel32.dll")]
private static extern IntPtr CreateFile(...);
```

**Compliant:**

```csharp
[DllImport("kernel32.dll")]
private static extern SafeFileHandle CreateFile(...);
```

---

## S3925: `ISerializable` should be implemented correctly

**Severity:** Critical  
**Description:** Implementing `ISerializable` requires a protected constructor and proper `GetObjectData` handling to avoid security issues.

**Noncompliant:**

```csharp
[Serializable]
public class CustomException : Exception
{
    public CustomException(string message) : base(message) { }
}
```

**Compliant:**

```csharp
[Serializable]
public class CustomException : Exception
{
    public CustomException(string message) : base(message) { }

    protected CustomException(SerializationInfo info, StreamingContext context)
        : base(info, context) { }
}
```

---

## S3994: `Uri.IsWellFormedUriString` should not be used

**Severity:** Major  
**Description:** `Uri.IsWellFormedUriString` returns `false` for valid URIs that contain internationalized domain names.

**Noncompliant:**

```csharp
public bool IsValid(string url)
{
    return Uri.IsWellFormedUriString(url, UriKind.Absolute);
}
```

**Compliant:**

```csharp
public bool IsValid(string url)
{
    return Uri.TryCreate(url, UriKind.Absolute, out _);
}
```

---

## S3996: `Task` should not be returned from `async` methods without awaiting

**Severity:** Major  
**Description:** Returning a `Task` from an `async` method without awaiting it causes the exception to be unobserved.

**Noncompliant:**

```csharp
public async Task DoWorkAsync()
{
    return Task.Run(() => { });
}
```

**Compliant:**

```csharp
public async Task DoWorkAsync()
{
    await Task.Run(() => { });
}
```

---

## S4000: Delegates should not be used as event handlers if they are not compatible

**Severity:** Major  
**Description:** Delegates with mismatched signatures used as event handlers cause runtime exceptions.

**Noncompliant:**

```csharp
public event Action<string> OnProcess;

public void Trigger()
{
    OnProcess += () => { }; // Signature mismatch
}
```

**Compliant:**

```csharp
public event Action<string> OnProcess;

public void Trigger()
{
    OnProcess += (message) => { };
}
```

---

## S4016: Generic type parameters should be named `T` or prefixed with `T`

**Severity:** Minor  
**Description:** Following the `T` prefix convention makes generic code easier to read.

**Noncompliant:**

```csharp
public class Container<Element>
{
}
```

**Compliant:**

```csharp
public class Container<TElement>
{
}
```

---

## S4027: Exceptions should not be thrown in finally blocks

**Severity:** Blocker  
**Description:** Throwing an exception in a `finally` block suppresses the original exception.

**Noncompliant:**

```csharp
public void Process()
{
    try
    {
        DoWork();
    }
    finally
    {
        throw new InvalidOperationException("Cleanup failed");
    }
}
```

**Compliant:**

```csharp
public void Process()
{
    try
    {
        DoWork();
    }
    finally
    {
        Cleanup();
    }
}
```

---

## S4039: `Delegate.Subtract` and `Delegate.Remove` should not be used in events

**Severity:** Major  
**Description:** Removing a multicast delegate removes all occurrences, not just the last added, which is usually not intended.

**Noncompliant:**

```csharp
eventHandler -= handler;
```

**Compliant:**

```csharp
// Use a collection or event accessor pattern for safe removal
```

---

## S4049: Method names should not be suffixed with `Async` if they are not async

**Severity:** Minor  
**Description:** The `Async` suffix implies the method returns a `Task` and is awaitable.

**Noncompliant:**

```csharp
public void LoadAsync()
{
    // Not async
}
```

**Compliant:**

```csharp
public async Task LoadAsync()
{
    await LoadDataAsync();
}
```

---

## S4056: Overloads with a `params` array should not be defined

**Severity:** Minor  
**Description:** Overloads that take an array and a `params` version can cause ambiguity and unexpected boxing.

**Noncompliant:**

```csharp
public void Log(string message) { }
public void Log(params string[] messages) { }
```

**Compliant:**

```csharp
public void Log(string message) { }
public void Log(IEnumerable<string> messages) { }
```

---

## S4058: Overloads should not differ only by `ref` or `out`

**Severity:** Major  
**Description:** Methods that differ only by `ref`/`out` parameters are confusing for callers and compilers.

**Noncompliant:**

```csharp
public void Process(int value) { }
public void Process(out int value) { value = 0; }
```

**Compliant:**

```csharp
public void Process(int value) { }
public bool TryProcess(out int value) { value = 0; return true; }
```

---

## S4060: `volatile` fields should not be used

**Severity:** Major  
**Description:** `volatile` does not provide atomicity for compound operations. Use `Interlocked` or `lock` instead.

**Noncompliant:**

```csharp
private volatile int _counter;

public void Increment()
{
    _counter++;
}
```

**Compliant:**

```csharp
private int _counter;

public void Increment()
{
    Interlocked.Increment(ref _counter);
}
```

---

## S4069: `Increment` and `Decrement` operators should not be used in a method call

**Severity:** Minor  
**Description:** Using `++` or `--` inside a method argument makes evaluation order unclear.

**Noncompliant:**

```csharp
public void Process(int[] values, int index)
{
    Console.WriteLine(values[index++]);
}
```

**Compliant:**

```csharp
public void Process(int[] values, int index)
{
    Console.WriteLine(values[index]);
    index++;
}
```

---

## S4142: Duplicate values should not be passed as arguments

**Severity:** Major  
**Description:** Passing the same value to two different parameters is usually a copy-paste mistake.

**Noncompliant:**

```csharp
public void Move(int x, int y)
{
    _cursor.Move(x, x);
}
```

**Compliant:**

```csharp
public void Move(int x, int y)
{
    _cursor.Move(x, y);
}
```

---

## S4158: Empty collections should not be passed as arguments

**Severity:** Major  
**Description:** Passing an empty collection literal or newly created empty collection is pointless.

**Noncompliant:**

```csharp
public void Process()
{
    SaveItems(new List<string>());
}
```

**Compliant:**

```csharp
public void Process(List<string> items)
{
    if (items.Count > 0)
    {
        SaveItems(items);
    }
}
```

---

## S4260: `nameof` should be used instead of string literals for parameter names

**Severity:** Major  
**Description:** String literals for parameter names are not refactor-safe and can become stale.

**Noncompliant:**

```csharp
public void Process(string value)
{
    ArgumentException.ThrowIfNullOrEmpty("value", value);
}
```

**Compliant:**

```csharp
public void Process(string value)
{
    ArgumentException.ThrowIfNullOrEmpty(nameof(value), value);
}
```

---

## S4277: `Shared` or `static` fields should be read-only when public

**Severity:** Major  
**Description:** Public mutable static fields can be changed by any code, leading to unexpected state changes.

**Noncompliant:**

```csharp
public static int MaxRetries = 3;
```

**Compliant:**

```csharp
public static readonly int MaxRetries = 3;
```

---

## S4487: Unread `private` fields should be removed

**Severity:** Major  
**Description:** Private fields that are never read indicate dead code or incomplete implementation.

**Noncompliant:**

```csharp
public class Processor
{
    private readonly ILogger<Processor> _logger;

    public Processor(ILogger<Processor> logger)
    {
        _logger = logger;
    }

    public void Process()
    {
        // Never uses _logger
    }
}
```

**Compliant:**

```csharp
public class Processor
{
    private readonly ILogger<Processor> _logger;

    public Processor(ILogger<Processor> logger)
    {
        _logger = logger;
    }

    public void Process()
    {
        _logger.LogInformation("Processing started");
    }
}
```

---

## S4507: Delivering code in debug mode should not be done

**Severity:** Major  
**Description:** Debug builds expose detailed error information and may disable security features.

**Noncompliant:**

```csharp
#if DEBUG
    Console.WriteLine("Debug info: " + password);
#endif
```

**Compliant:**

```csharp
// Remove all DEBUG-only code before release builds
```

---

## S4792: Configuring loggers is security-sensitive

**Severity:** Major  
**Description:** Misconfigured logging can expose sensitive data. Never log secrets, passwords, or PII.

**Noncompliant:**

```csharp
_logger.LogInformation("User logged in with password {Password}", password);
```

**Compliant:**

```csharp
_logger.LogInformation("User {UserId} logged in", userId);
```

---

## S5679: Regular expressions should not contain empty groups

**Severity:** Minor  
**Description:** Empty groups in regex serve no purpose and indicate a mistake.

**Noncompliant:**

```csharp
var pattern = @"hello()world";
```

**Compliant:**

```csharp
var pattern = @"helloworld";
```

---

## S6359: `ThreadStatic` fields should not be used with `async` methods

**Severity:** Major  
**Description:** `async` methods may resume on different threads, making `ThreadStatic` values unreliable.

**Noncompliant:**

```csharp
[ThreadStatic]
private static int _counter;

public async Task ProcessAsync()
{
    _counter++;
    await Task.Delay(100);
    Console.WriteLine(_counter); // May be wrong thread
}
```

**Compliant:**

```csharp
private static readonly AsyncLocal<int> _counter = new();

public async Task ProcessAsync()
{
    _counter.Value++;
    await Task.Delay(100);
    Console.WriteLine(_counter.Value);
}
```

---

# Security Hotspots

Rules that identify security-sensitive code that requires manual review.

## S2068: Credentials should not be hard-coded

**Severity:** Blocker  
**Description:** Hardcoded passwords, API keys, or connection strings are visible in source control and binaries.

**Noncompliant:**

```csharp
public const string ApiKey = "sk-1234567890abcdef";
```

**Compliant:**

```csharp
public class ApiOptions
{
    public required string ApiKey { get; init; }
}
```

---

## S2077: SQL injection risks should be mitigated

**Severity:** Blocker  
**Description:** Concatenating user input into SQL queries creates injection vulnerabilities. Use parameterized queries.

**Noncompliant:**

```csharp
public async Task<List<User>> Search(string name)
{
    var sql = $"SELECT * FROM Users WHERE Name = '{name}'";
    return await _context.Users.FromSqlRaw(sql).ToListAsync();
}
```

**Compliant:**

```csharp
public async Task<List<User>> Search(string name)
{
    return await _context.Users
        .Where(u => u.Name == name)
        .ToListAsync();
}
```

---

## S2091: XPath injection risks should be mitigated

**Severity:** Blocker  
**Description:** User input in XPath expressions can alter query logic. Validate and parameterize inputs.

**Noncompliant:**

```csharp
public string FindUser(string username)
{
    var xpath = $"//user[name='{username}']";
    return _document.SelectSingleNode(xpath)?.InnerText;
}
```

**Compliant:**

```csharp
public string FindUser(string username)
{
    var navigator = _document.CreateNavigator();
    var expr = navigator.Compile("//user[name=$name]");
    var resolver = new XPathExpressionResolver();
    resolver.AddParam("name", username);
    return navigator.SelectSingleNode(expr, resolver)?.InnerText;
}
```

---

## S2245: Random values should not be used for security purposes

**Severity:** Blocker  
**Description:** `System.Random` is deterministic and predictable. Use `RandomNumberGenerator` for security-sensitive operations.

**Noncompliant:**

```csharp
public string GeneratePassword()
{
    var random = new Random();
    return random.Next().ToString();
}
```

**Compliant:**

```csharp
public string GeneratePassword()
{
    var bytes = new byte[32];
    RandomNumberGenerator.Fill(bytes);
    return Convert.ToBase64String(bytes);
}
```

---

## S2257: Using weak hash algorithms is security-sensitive

**Severity:** Critical  
**Description:** MD5 and SHA-1 are vulnerable to collision attacks. Use SHA-256 or stronger for hashing.

**Noncompliant:**

```csharp
public byte[] HashPassword(string password)
{
    using var md5 = MD5.Create();
    return md5.ComputeHash(Encoding.UTF8.GetBytes(password));
}
```

**Compliant:**

```csharp
public byte[] HashPassword(string password)
{
    using var sha256 = SHA256.Create();
    return sha256.ComputeHash(Encoding.UTF8.GetBytes(password));
}
```

---

## S2612: Prohibited classes should not be used

**Severity:** Major  
**Description:** Certain classes like `BinaryFormatter` are dangerous due to deserialization vulnerabilities and should not be used.

**Noncompliant:**

```csharp
var formatter = new BinaryFormatter();
formatter.Serialize(stream, obj);
```

**Compliant:**

```csharp
// Use System.Text.Json or MessagePack instead
var json = JsonSerializer.Serialize(obj);
```

---

## S3011: Bypassing accessibility is security-sensitive

**Severity:** Major  
**Description:** Using reflection to access non-public members breaks encapsulation and can expose internal state.

**Noncompliant:**

```csharp
var field = typeof(User).GetField("_password", BindingFlags.NonPublic | BindingFlags.Instance);
var password = field?.GetValue(user);
```

**Compliant:**

```csharp
// Use public properties and methods only
var password = user.GetPasswordHash();
```

---

## S3330: Server-side requests should not be vulnerable to SSRF attacks

**Severity:** Blocker  
**Description:** SSRF occurs when user-controlled URLs are used to make server-side requests without validation.

**Noncompliant:**

```csharp
public async Task<string> Fetch(string url)
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);
}
```

**Compliant:**

```csharp
public async Task<string> Fetch(string url)
{
    var allowedHosts = new[] { "api.example.com", "cdn.example.com" };
    if (!Uri.TryCreate(url, UriKind.Absolute, out var uri)
        || !allowedHosts.Contains(uri.Host))
    {
        throw new ArgumentException("URL not allowed");
    }

    using var client = new HttpClient();
    return await client.GetStringAsync(uri);
}
```

---

## S4502: Disabling CSRF protections is security-sensitive

**Severity:** Blocker  
**Description:** Disabling CSRF validation in ASP.NET Core exposes the application to cross-site request forgery attacks.

**Noncompliant:**

```csharp
services.AddControllersWithViews(options =>
{
    options.Filters.Add(new IgnoreAntiforgeryTokenAttribute());
});
```

**Compliant:**

```csharp
services.AddControllersWithViews();
```

---

## S4790: Hashing data is security-sensitive

**Severity:** Critical  
**Description:** The choice of hash algorithm affects security. Use SHA-256 or stronger and avoid MD5/SHA-1.

**Noncompliant:**

```csharp
public byte[] Hash(string input)
{
    using var md5 = MD5.Create();
    return md5.ComputeHash(Encoding.UTF8.GetBytes(input));
}
```

**Compliant:**

```csharp
public byte[] Hash(string input)
{
    using var sha256 = SHA256.Create();
    return sha256.ComputeHash(Encoding.UTF8.GetBytes(input));
}
```

---

## S4792: Configuring loggers is security-sensitive

**Severity:** Major  
**Description:** Logger configuration that includes sensitive fields or writes to insecure destinations can leak data.

**Noncompliant:**

```csharp
_logger.LogInformation("Login: {Email} {Password}", email, password);
```

**Compliant:**

```csharp
_logger.LogInformation("User {UserId} logged in", userId);
```

---

## S5324: `Random` should not be used for security-sensitive purposes

**Severity:** Blocker  
**Description:** `Random` is not cryptographically secure and should not be used for tokens, keys, or passwords.

**Noncompliant:**

```csharp
public string GenerateToken()
{
    var random = new Random();
    return random.Next().ToString();
}
```

**Compliant:**

```csharp
public string GenerateToken()
{
    var bytes = new byte[32];
    RandomNumberGenerator.Fill(bytes);
    return Convert.ToHexString(bytes);
}
```

---

## S5332: Using clear-text protocols is security-sensitive

**Severity:** Blocker  
**Description:** Sending data over HTTP or FTP exposes it to interception. Use HTTPS and SFTP.

**Noncompliant:**

```csharp
using var client = new HttpClient();
return await client.GetStringAsync("http://api.example.com/data");
```

**Compliant:**

```csharp
using var client = new HttpClient();
return await client.GetStringAsync("https://api.example.com/data");
```

---

## S5443: Using public writable directories is security-sensitive

**Severity:** Major  
**Description:** Writing files to world-writable directories can allow other users to modify or replace them.

**Noncompliant:**

```csharp
var path = "/tmp/data.json";
File.WriteAllText(path, json);
```

**Compliant:**

```csharp
var path = Path.Combine(Path.GetTempPath(), Guid.NewGuid().ToString() + ".json");
File.WriteAllText(path, json);
```

---

## S5445: Using temporary files is security-sensitive

**Severity:** Major  
**Description:** Temporary files may have weak permissions or be predictable. Use secure APIs and explicit cleanup.

**Noncompliant:**

```csharp
var path = "/tmp/tempfile.txt";
File.WriteAllText(path, data);
```

**Compliant:**

```csharp
var path = Path.GetTempFileName();
try
{
    File.WriteAllText(path, data);
}
finally
{
    File.Delete(path);
}
```

---

## S5659: SSL/TLS connection settings should not be bypassed

**Severity:** Blocker  
**Description:** Disabling certificate validation allows man-in-the-middle attacks. Never bypass validation in production.

**Noncompliant:**

```csharp
ServicePointManager.ServerCertificateValidationCallback = (sender, cert, chain, errors) => true;
```

**Compliant:**

```csharp
// Use default validation
```

---

## S5689: Deserializing objects from an untrusted source is security-sensitive

**Severity:** Blocker  
**Description:** Deserializing untrusted data can lead to remote code execution. Use safe serializers and whitelist types.

**Noncompliant:**

```csharp
var formatter = new BinaryFormatter();
var obj = formatter.Deserialize(stream);
```

**Compliant:**

```csharp
var obj = JsonSerializer.Deserialize<MyDto>(json);
```

---

## S5693: Allowing both HTTP and HTTPS is security-sensitive

**Severity:** Major  
**Description:** Allowing HTTP alongside HTTPS exposes traffic to interception. Redirect HTTP to HTTPS.

**Noncompliant:**

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
});
```

**Compliant:**

```csharp
app.UseHttpsRedirection();
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
});
```

---

## S5766: Deserializing with `TypeNameHandling.All` is security-sensitive

**Severity:** Blocker  
**Description:** `TypeNameHandling.All` in Newtonsoft.Json allows arbitrary type instantiation during deserialization.

**Noncompliant:**

```csharp
var settings = new JsonSerializerSettings
{
    TypeNameHandling = TypeNameHandling.All
};
var obj = JsonConvert.DeserializeObject<MyType>(json, settings);
```

**Compliant:**

```csharp
var obj = JsonConvert.DeserializeObject<MyType>(json);
```

---

## S6244: Using weak SSL/TLS protocols is security-sensitive

**Severity:** Blocker  
**Description:** Enabling TLS 1.0 or 1.1 exposes the application to known attacks. Use TLS 1.2 or 1.3.

**Noncompliant:**

```csharp
var handler = new HttpClientHandler
{
    SslProtocols = SslProtocols.Tls | SslProtocols.Tls11
};
```

**Compliant:**

```csharp
var handler = new HttpClientHandler
{
    SslProtocols = SslProtocols.Tls12 | SslProtocols.Tls13
};
```

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
13. Ensure generated code complies with all SonarQube rules defined in this document.
14. Do not generate code that triggers SonarQube bugs, vulnerabilities, or critical code smells.
15. Apply security hotspot guidance for any security-sensitive operations (cryptography, serialization, networking, logging).
