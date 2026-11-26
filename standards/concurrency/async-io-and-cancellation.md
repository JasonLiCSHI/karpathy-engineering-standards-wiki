---
title: Async I/O and Cancellation
category: concepts
tags: [csharp, concurrency, cancellation, visibility/public]
sources: [distilled-engineering-standards]
summary: Standard for async I/O and cancellation in C# service code.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
relationships:
  - target: "[[standards/concurrency/csharp-asynchronous-programming-practices]]"
    type: related_to
  - target: "[[standards/concurrency/concurrent-programming-practices]]"
    type: related_to
tier: supporting
---

# Async I/O and Cancellation

This section defines the repository's expectation for asynchronous I/O and cancellation flow in modern .NET. These patterns are the default for service boundaries, network calls, persistence, and long-running workflows.

## 1. Default pattern

Prefer this structure:

```csharp
public async Task<Result> LoadAsync(string id, CancellationToken cancellationToken)
{
    cancellationToken.ThrowIfCancellationRequested();

    var result = await _repository.GetByIdAsync(id, cancellationToken);
    return result;
}
```

Rules:

- pass `CancellationToken` through all async layers
- do not create hidden cancellation boundaries
- do not swallow cancellation unless the API contract explicitly calls for it
- keep async methods readable and linear

## 2. Public boundary behavior

At public service boundaries, cancellation should be treated as a first-class API contract:

```csharp
public async Task<Study> GetStudyAsync(Guid studyId, CancellationToken cancellationToken)
```

This should be used for:

- HTTP controller action methods
- repository calls
- queue consumers
- long-running pipeline steps
- worker jobs and background services

Do not allow public APIs to accept a `CancellationToken` only in the implementation and silently ignore it.

## 3. Cancellation semantics

Cancellation is not the same as failure.

Use cancellation for:

- user aborts
- timeout handling
- shutdown requests
- external stop signals

Use exceptions for:

- validation failures
- dependency failure
- data integrity problems
- unexpected runtime errors

When catching `OperationCanceledException`, preserve the semantics:

```csharp
catch (OperationCanceledException) when (cancellationToken.IsCancellationRequested)
{
    throw;
}
```

## 4. Async I/O rule

Perform I/O through async APIs:

- `HttpClient.GetAsync`
- EF Core async queries
- `Stream.ReadAsync` / `WriteAsync`
- database drivers and queue clients with async APIs

Avoid:

- `.Result`
- `.Wait()`
- synchronous `Read/Write` calls inside service or API code
- wrapping async code in a blocking adapter for convenience

## 5. Background processing and timeouts

Background services should honor shutdown signals and observation boundaries.

Prefer:

- `BackgroundService`-style loops
- `PeriodicTimer` for scheduled checks
- bounded queue workers
- worker-level timeouts for external calls

Avoid:

- infinite loops without cancellation
- fire-and-forget tasks without a defined lifecycle
- unbounded retries without backoff and detection

## 6. When to avoid async

The async model is not a universal answer. Do not convert trivial synchronous code simply to be "modern". Async is best when the method truly contains asynchronous waiting or external I/O.

Avoid async for:

- tiny computations with no waiting
- operations that do not block on I/O or external dependencies
- wrappers that only call a synchronous API and then immediately `await` it

## 7. Timeout and retry design

When systems depend on external services, use explicit timeout and retry policy:

- configure per dependency
- keep retry backoff bounded
- avoid retry storms under load
- distinguish transient from permanent failures

This should be policy-aware, not ad hoc.

## 8. Recommended standard

The repository should default to:

- async I/O for all external dependencies
- cancellation tokens on all async service methods
- explicit timeout and retry behavior for remote dependencies
- no blocking waits inside async flows
- no `async void` except framework event callbacks
