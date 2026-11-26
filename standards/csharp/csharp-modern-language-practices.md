---
title: C# Modern Language Practices
category: skills
tags: [csharp, modern-csharp, language-version, code-review, visibility/public]
sources:
  - https://learn.microsoft.com/dotnet/csharp/whats-new/csharp-9
  - https://learn.microsoft.com/dotnet/csharp/whats-new/csharp-10
  - https://learn.microsoft.com/dotnet/csharp/whats-new/csharp-11
  - https://learn.microsoft.com/dotnet/csharp/whats-new/csharp-12
  - https://learn.microsoft.com/dotnet/csharp/whats-new/csharp-13
  - https://learn.microsoft.com/dotnet/csharp/whats-new/csharp-14
summary: "A practical repo standard for .NET 10 / C# 14-era coding: use modern features when they improve safety, clarity, and correctness, not merely because they are new."
provenance:
  extracted: 0.83
  inferred: 0.17
  ambiguous: 0.00
base_confidence: 0.94
lifecycle: verified
lifecycle_changed: 2026-08-21
tier: core
created: 2026-08-21T15:01:00+08:00
updated: 2026-08-21T15:15:00+08:00
relationships:
  - target: "[[standards/csharp/modern-csharp-standard]]"
    type: extends
  - target: "[[standards/csharp/csharp-language-version-aware-coding]]"
    type: related_to
---

# C# Modern Language Practices

## Goal

This note is the project-level companion to [[standards/csharp/csharp-coding-standard]] and [[standards/csharp/csharp-language-version-aware-coding]]. Its purpose is to define the repository baseline for modern C# from C# 9 through the current .NET 10 / C# 14 era.

The rule is simple:

- prefer the newest stable feature when it materially improves correctness, safety, or clarity;
- do not use novelty syntax just to impress readers;
- do not require every file to use every new feature.

## Core principle

For this codebase, the effective standard is:

- use the language version supported by the project and target framework;
- prefer immutable and explicit models where the domain is value-like;
- use pattern matching and structured branching when they are easier to read than nested conditionals;
- prefer `required`, `init`, and record semantics to reduce invalid object states;
- keep preview features out of shared code unless the project explicitly approves them.

## C# 9 baseline

C# 9 changed the default shape of modern code in a durable way.

### `record`

Use records for immutable data, messages, DTOs, snapshots, and domain values that should be compared by value.

```csharp
public record CreateStudyCommand
{
    public required string StudyId { get; init; }
    public required string PatientId { get; init; }
    public DateTimeOffset RequestedAt { get; init; } = DateTimeOffset.UtcNow;
}
```

Prefer records when:

- equality should be structural instead of reference-based;
- the object is a data carrier not a mutable lifecycle entity;
- copying or snapshot semantics are part of the design.

Avoid converting mutable entity aggregates into records just for brevity.

### `init` and with-expressions

Use `init` for object construction patterns that are intentionally immutable after setup.

```csharp
var request = new CreateStudyCommand
{
    StudyId = studyId,
    PatientId = patientId
};
```

This reduces invalid partially initialized states and makes the creation semantics obvious.

### Pattern matching and target-typed `new`

Prefer improved pattern matching and target-typed construction when the statement becomes easier to read.

```csharp
var result = value switch
{
    { Status: StudyStatus.Ready } => "ready",
    { Status: StudyStatus.Running } => "running",
    _ => "unknown"
};
```

## C# 10 and beyond

### File-scoped namespaces and stronger expressiveness

Use file-scoped namespaces when the project style already supports them and the file layout remains clear.

### `required` members

Require object fields that are mandatory for correct creation.

```csharp
public sealed class ServiceOptions
{
    public required string ConnectionString { get; init; }
    public required string ServiceName { get; init; }
}
```

This is particularly useful for configuration objects and DTO boundaries.

### `record struct` and value semantics

Use small value types when they encode a domain concept and are meant to behave as a compact value.

## C# 11 to C# 12

### Raw strings and collection expressions

Prefer raw strings for multi-line embedded text and templates.

```csharp
var sql = """
SELECT *
FROM Study
WHERE StudyId = {studyId}
""";
```

Prefer collection expressions when they make initialization easier to read:

```csharp
string[] tags = ["backend", "csharp", "daily"];
```

### Primary constructors

Use primary constructors for small orchestration objects or DTOs when the constructor is not hiding meaningful domain logic. Do not use them to compress complex business objects into unclear or opaque code.

## C# 13 and C# 14

Treat newer features as options, not as mandatory style. The rule is still:

- use the feature when it improves correctness or readability;
- avoid preview-only features in shared code unless explicitly approved;
- keep the repository coherent and understandable for code reviewers.

## Anti-patterns

Avoid these habits:

- using a new feature just because it exists;
- replacing simple code with dense syntax that reduces readability;
- converting mutable aggregates into records without a domain reason;
- using preview features in shared libraries without a migration note or explicit sign-off;
- increasing abstraction for the sake of language novelty.

## Repository standard

As a modern .NET baseline, the recommended project default is:

- use records and `init` for immutable payloads and snapshot-like contracts;
- prefer `required` for mandatory configuration and DTO fields;
- prefer pattern matching and switch expressions when they make the logic clearer;
- prefer raw strings and collection expressions when the result is more readable;
- keep modern language features aligned with the actual compiler version and repo conventions;
- never let syntax modernization weaken correctness, API clarity, or reviewer confidence.

## Related notes

- [[standards/csharp/csharp-coding-standard]]
- [[standards/csharp/csharp-language-version-aware-coding]]
- [[standards/concurrency/csharp-asynchronous-programming-practices]]
- [[standards/concurrency/concurrent-programming-practices]]
- [[standards/performance/csharp-performance-practices]]
