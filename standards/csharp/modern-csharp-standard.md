---
title: Modern C# Programming Standard
category: conventions
tags: [csharp, modern-csharp, visibility/public]
sources: [distilled-engineering-standards]
summary: Modern C# programming standard for the repository.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: core
relationships:
  - target: "[[standards/csharp/csharp-coding-standard]]"
    type: related_to
  - target: "[[standards/csharp/csharp-modern-language-practices]]"
    type: related_to
  - target: "[[standards/csharp/csharp-language-version-aware-coding]]"
    type: related_to
---

# Modern C# Programming Standard

This document defines the repository's baseline for modern C# programming, from C# 9 to the current .NET 10 / C# 14-era language model. The standard is intended to be practical and opinionated: it favors readability, safety, and domain clarity over "clever" syntax.

## 1. Scope and principles

Modern C# should be used to improve the following qualities:

- correctness and null safety
- model clarity for domain objects
- readability in business logic and service code
- maintainability across long-lived codebases
- consistency across backend and cross-platform code

The standard does not require every new feature to be used everywhere. It requires the team to prefer the feature that clearly matches the problem and the project's maturity.

## 2. C# version map and project stance

The repository currently targets modern .NET and should treat the following language evolution as the default baseline:

### C# 9

Key features that are now part of the standard:

- `record` types for immutable data and value-based equality
- `init` setters for object creation patterns
- `with` expressions for non-destructive updates
- improved pattern matching
- target-typed `new` expressions
- top-level statements for very small scripts and tooling, but not for production library code

### C# 10

Key features that are expected to be used when appropriate:

- file-scoped namespaces to reduce boilerplate
- `record struct` and struct-focused value objects
- `required` members to enforce object creation contracts
- better interpolation and cleaner code organization patterns
- general immutability-oriented modeling

### C# 11

Key features to prefer when they increase clarity:

- raw string literals for embedded content and test data
- list patterns for expressive matching
- generic attributes and more type-safe APIs
- file-local types for internal-only helper types
- structured pattern matching for complex data shapes

### C# 12

Key features to adopt selectively:

- primary constructors for lightweight DTOs or orchestration objects
- collection expressions for array/list initialization improvements
- more concise and expressive code in internal composition code

### C# 13 and C# 14

This repo should treat newer features as modern options, not as required language style. For example:

- preview features must be explicitly approved before adoption
- use the current .NET 10 SDK as the authority for stable syntax
- do not use preview-only features in shared libraries without a clear migration plan

The important rule is not "use the newest syntax"; it is "use the correct feature for the correct abstraction".

## 3. Preferred patterns

### 3.1 Prefer immutable domain models

Use `record` and `init` for DTOs, command payloads, API contracts, and domain value objects when mutation is not required.

```csharp
public record CreateStudyCommand
{
    public required string StudyId { get; init; }
    public required string PatientId { get; init; }
    public DateTimeOffset RequestedAt { get; init; } = DateTimeOffset.UtcNow;
}
```

Why:

- clearer intent than mutable POCOs
- safer for concurrent and long-lived workflows
- better equality semantics for domain values
- easier to reason about with `with` expressions

Use mutable state only where object lifecycle and mutation are truly required, such as internal runtime state or streaming pipelines.

### 3.2 Use `required` instead of silent nulls

For public DTOs, API payloads, and configuration models, use `required` to ensure creation-time correctness.

```csharp
public sealed class ServiceOptions
{
    public required string ConnectionString { get; init; }
    public required string ServiceName { get; init; }
}
```

Do not rely on optional values that later fail far away from the call site. If a property is required, encode that in the type.

### 3.3 Prefer `record` for value equality and snapshot semantics

Records are the preferred choice when representing business values that should be compared by contents rather than by object identity.

```csharp
public record PatientIdentifier(string Value);
```

This is especially useful for domain IDs, API contracts, and messages that are meant to be copied, compared, and passed around safely.

### 3.4 Prefer `switch` expressions and pattern matching over chained conditionals

Modern C# patterns should replace verbose nested `if` logic when the structure is a typed decision model.

```csharp
var status = order switch
{
    { State: OrderState.Queued } => "queued",
    { State: OrderState.InProgress } => "running",
    { State: OrderState.Completed } => "done",
    _ => "unknown"
};
```

Prefer:

- `switch` expressions
- `is` pattern checks
- property patterns
- list patterns when matching collections

Avoid:

- long nested condition trees with manual casts
- repeated `if (x is SomeType some)` followed by a large amount of null-check boilerplate

### 3.5 Use `using` declarations and `await using` consistently

Prefer the modern disposal style for readability and reduced nesting.

```csharp
using var connection = await factory.CreateConnectionAsync(token);
await using var stream = file.OpenRead();
```

This is especially useful for short-lived dependencies, IO resource cleanup, and service-scoped resources.

### 3.6 Keep constructors clear and minimal

Use constructors to express required dependencies and required creation state, rather than hiding dependency setup in complex initialization methods.

```csharp
public sealed class StudyService
{
    private readonly IStudyRepository _repository;
    private readonly ILogger<StudyService> _logger;

    public StudyService(IStudyRepository repository, ILogger<StudyService> logger)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
}
```

When the project allows it, primary constructors may be used for lightweight orchestration types, but they must not hide business complexity or confuse domain intent.

## 4. Nullability and safety rules

The codebase should treat nullable reference types as a required safety mechanism, not an optional style preference.

### Rules

- never use nullable annotations as a substitute for business validation
- use `ArgumentNullException.ThrowIfNull(...)` or `ArgumentException` for defensive checks
- prefer `string?` and `T?` when the value is genuinely optional
- avoid returning `null` from public APIs unless the domain explicitly requires it
- use `required`, `init`, and immutability to reduce null burden in object graphs

```csharp
public sealed class StudyRequest
{
    public required string StudyId { get; init; }
    public string? Notes { get; init; }
}
```

If an input is invalid, fail fast at the boundary; do not allow `null` to propagate silently through the application.

## 5. String handling and literals

### Use interpolated strings for formatted output

```csharp
_logger.LogInformation("Study {StudyId} completed in {ElapsedMs} ms", studyId, elapsedMs);
```

### Use raw string literals for multi-line templates and embedded examples

```csharp
var sql = """
SELECT *
FROM Study
WHERE StudyId = {studyId}
""";
```

This is especially useful for SQL templates, geometry, configuration snippets, and test fixtures. Keep the contents readable and avoid overuse for simple one-line strings.

## 6. Collection and data construction

Modern C# has made collection creation more concise and expressive. Use collection expressions when they improve readability and do not hide intent.

```csharp
var ids = ["s1", "s2", "s3"];
var values = [..existingValues, newValue];
```

Prefer:

- `[]` for empty collections when the type is clear
- `[..existing]` if the code is clearer than repeated `.AddRange()` calls
- strongly typed collections where the business logic depends on type semantics

Do not convert a small list to a complex expression just to use the newest syntax. Clarity comes before syntactic novelty.

## 7. Asynchronous and cancellation-safe code

This repo should continue to prefer async/await with `CancellationToken` propagation.

Rules:

- every long-running or IO-bound operation should accept a `CancellationToken`
- cancellation should flow through the call stack
- do not swallow cancellation exceptions except at a known boundary
- use `ConfigureAwait(false)` only when the project explicitly requires it and it matches the repository pattern

```csharp
public async Task<Study> GetStudyAsync(string studyId, CancellationToken cancellationToken)
{
    cancellationToken.ThrowIfCancellationRequested();
    return await _repository.GetAsync(studyId, cancellationToken);
}
```

## 8. File and namespace organization

Prefer modern file organization patterns for small, focused code files.

- use file-scoped namespaces in new code when consistent with repo patterns
- keep each file responsible for one primary type
- use `global using` only when it adds value and does not obscure required dependencies
- avoid broad namespace pollution in shared library code

For the repo, prefer explicitness over magic global imports. This keeps the dependency surface understandable during reviews.

## 9. Public API design

Modern C# encourages public API design that is explicit and stable.

Rules:

- public contracts should be as simple and explicit as possible
- prefer `record` and `readonly` value types where semantics are value-like
- avoid exposing mutable shared collections directly
- use `IReadOnlyList<T>` or `IReadOnlyCollection<T>` for read-only APIs
- do not widen APIs unnecessarily just to use a newer language feature

```csharp
public sealed record StudySnapshot(
    string StudyId,
    string PatientId,
    IReadOnlyList<string> Tags);
```

## 10. When not to use newer features

Do not use a new C# feature just because it is available. Avoid it when:

- the code becomes harder to read
- the feature is preview-only and not clearly approved
- the team cannot explain the runtime semantics to reviewers
- the code is in a hot path and the feature adds complexity without measurable value
- the repository's coding pattern and style does not support the syntax

The modern-language rule is: prefer correctness, readability, and transparency over cleverness.

## 11. Practical review checklist

Before merging a change, confirm that:

- immutable models are preferred unless mutation is required
- `record`, `init`, and `required` are used where they reduce ambiguity
- public APIs are explicit and strongly typed
- pattern matching is used instead of nested `if` chains when it improves clarity
- nullability is handled at the boundary, not hidden in the middle of the stack
- `CancellationToken` is propagated in async code
- collection and string syntax are readable and idiomatic
- preview features are not introduced without approval

## 12. Recommended standard for this repository

At a project level, the repository should follow this default rule set:

- prefer `record` and `required` in DTOs and messages
- prefer `init` for immutable object creation
- prefer pattern matching for typed branching
- prefer structured logging and explicit validation over ad hoc object mutation
- use `async`/`await` + `CancellationToken` consistently
- prefer file-scoped namespaces and modern code organization when already aligned with the repo's style
- avoid preview-only features in shared code without a project-level decision

This gives the team a modern C# baseline that is compatible with .NET 10 / C# 14 while staying explicit, production-safe, and maintainable.
