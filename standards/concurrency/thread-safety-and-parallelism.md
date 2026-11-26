---
title: Thread Safety and Parallelism
category: concepts
tags: [csharp, concurrency, thread-safety, visibility/public]
sources: [distilled-engineering-standards]
summary: Standard for thread safety and parallelism in C# code.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
relationships:
  - target: "[[standards/concurrency/concurrent-programming-practices]]"
    type: related_to
tier: supporting
---

# Thread Safety and Parallelism

This section defines the repository's rules for safe multithreading and bounded parallelism in .NET. Threading is permitted only where it is explicit, justified, and controlled.

## 1. Correctness comes before throughput

The first rule of concurrency is that correctness trumps throughput. A small, safe, single-threaded path is preferred over a faster but race-prone implementation.

When concurrency is introduced, the code must answer:

- what state is shared
- what synchronization mechanism is used
- how failures and cancellation are handled
- what the expected bounded concurrency is

## 2. Shared mutable state must be controlled

Shared state is the root of most concurrency bugs.

Examples of shared mutable state:

- caches
- service-scoped dictionaries
- collection snapshots reused across tasks
- singleton objects with mutable data
- business state passed between worker tasks

Prefer:

- immutable data models
- per-call state instead of shared state
- `ConcurrentDictionary` or thread-safe collections when needed
- explicit producer/consumer pipelines

## 3. Use the right synchronization primitive

The repo should prefer the simplest correct tool.

### `lock`

Use when a short critical section is required and the scope is local.

### `SemaphoreSlim`

Use for async coordination and admission control.

### `Channel<T>`

Use for pipelined producer/consumer patterns and async queues.

### `ConcurrentDictionary<TKey, TValue>`

Use for thread-safe dictionary updates when the lifetime semantics are clear.

Avoid:

- broad locking around large operations
- locking across I/O and remote calls
- mixing lock styles without a clear purpose

## 4. Avoid deadlocks and lock inversion

These are some of the most expensive concurrency bugs.

Avoid:

- `Task.Wait()` or `.Result` while holding a lock
- locking while awaiting asynchronous code
- nested broad locking across multiple components
- re-entrant locking patterns without clear ownership

If a lock is needed, ensure the lock scope is short and the code path is predictable.

## 5. Parallelism must be bounded

Parallelism should be limited by design.

Good patterns:

- max worker count limits
- queue-backed processing
- bounded `Task.WhenAll` for known finite work
- `Parallel.ForEachAsync` with explicit degree of parallelism

Bad patterns:

- unbounded fan-out tasks for huge collections
- spawning a background task for every item without back-pressure
- allowing queue depth to grow without limits

## 6. Prefer async queues over fire-and-forget work

Do not use fire-and-forget tasks as a general service pattern.

Prefer:

- `Channel<T>` producer/consumer queues
- durable background job systems
- worker services with explicit lifecycle and telemetry
- retry and dead-letter handling for message processing

Background work should be observable and manageable.

## 7. CPU-bound work needs explicit design

CPU-bound work should only be parallelized when it is truly compute-heavy and measurable.

Examples:

- image processing
- compression and decompression
- large scientific or protocol transforms
- repeated computational analysis

Use limited parallelism and carefully measure the benefit. Do not parallelize CPU work by default.

## 8. Observability for concurrency issues

When concurrency is intentionally used, the code should expose enough telemetry for diagnosis.

Capture:

- item count and queue depth
- worker count and throttle settings
- timings for each phase of processing
- retry counts and cancellation events
- dead-letter or failure counts

This makes race conditions and queue stall issues diagnosable.

## 9. Recommended standard

The repository should default to:

- no manual thread creation unless a specific runtime need exists
- bounded parallelism only when there is a proven benefit
- explicit synchronization at the shared-state boundary
- pipeline-based async processing for heavy or queued work
- observability and cancellation in every concurrent workflow
