---
title: C# Performance Practices
category: skills
tags: [csharp, performance, span, memory, low-allocation, profiling, visibility/public]
sources:
  - https://learn.microsoft.com/dotnet/standard/memory-and-spans/overview
  - https://learn.microsoft.com/dotnet/standard/garbage-collection/garbage-collection-fundamentals
summary: Performance code in modern .NET should be explicit, well-measured, and low-allocation by default when the hot path justifies it.
provenance:
  extracted: 0.88
  inferred: 0.12
  ambiguous: 0.00
base_confidence: 0.93
lifecycle: verified
lifecycle_changed: 2026-08-21
tier: core
created: 2026-08-21T15:20:00+08:00
updated: 2026-08-21T15:28:00+08:00
relationships:
  - target: "[[standards/performance/low-allocation-and-span]]"
    type: related_to
  - target: "[[standards/performance/benchmarking-and-profiling]]"
    type: related_to
---

# C# Performance Practices

## Principle

Performance work in C# is not a blanket requirement to optimize every method. The correct standard is:

- measure first;
- optimize hot paths only;
- prefer clarity and correctness before micro-optimization;
- reduce allocations where they are measurable and meaningful;
- use modern .NET primitives such as `Span<T>`, `ReadOnlySpan<T>`, `Memory<T>`, pooling, and value-based processing when they align with the workload.

## Use `Span<T>` and `ReadOnlySpan<T>` when appropriate

Use spans for high-frequency parsing, buffer processing, and string/array slicing where keeping memory off the heap matters.

```csharp
public static int CountWhitespace(ReadOnlySpan<char> text)
{
    var count = 0;
    foreach (var ch in text)
    {
        if (char.IsWhiteSpace(ch))
            count++;
    }

    return count;
}
```

Prefer spans when:

- the logic works over a contiguous region of memory;
- the method is hot and allocation-sensitive;
- the operation is parse-oriented or protocol-oriented.

Avoid spans when:

- the code becomes less readable for a non-hot path;
- ownership and lifetime become confusing;
- syntax complexity is not justified by performance needs.

## Prefer `Memory<T>` when lifetime is important

`Memory<T>` is useful when a caller needs to retain a slice beyond the scope of the original buffer.

```csharp
public sealed class Parser
{
    private readonly Memory<byte> _buffer;

    public Parser(Memory<byte> buffer)
    {
        _buffer = buffer;
    }
}
```

Use `Memory<T>` to keep the abstraction safe when the underlying data might outlive the stack frame, but do not over-engineer this for ordinary in-memory business logic.

## Avoid unnecessary allocations

Allocation-heavy code is often the real performance problem. Common wins:

- avoid repeated string concatenation in loops;
- avoid `new` inside hot loops when a buffer or reusable object can be reused;
- prefer `string.Create`, `ArrayPool<T>`, and pooled buffers for large repeated operations;
- avoid LINQ in extremely hot paths when a simple loop is clearer and faster;
- avoid capturing closures in repeated asynchronous or scheduling work if the closure creates hidden allocations.

## `ArrayPool<T>` and pooled buffers

Use pooling when the operation is high frequency and buffer sizes vary or are large.

```csharp
var buffer = ArrayPool<byte>.Shared.Rent(4096);
try
{
    // use buffer
}
finally
{
    ArrayPool<byte>.Shared.Return(buffer);
}
```

This is particularly useful for network parsing, protocol reading, compression, or any repeated buffer lifecycle in hot paths.

## Measure and profile before optimizing

Do not optimize by assumption. Put the following rule in every hot path review:

1. benchmark the current behavior;
2. identify the bottleneck with profiling or metrics;
3. apply the smallest safe optimization;
4. re-measure and compare.

For C# code, common profiling signals include:

- allocation rate;
- Gen 0/1/2 collections;
- CPU hotspots in parser and serializer code;
- long-tail latency under load;
- lock contention and blocking calls.

## Reduce `Task` and async overhead when it is not needed

Async is excellent for I/O waiting and concurrency, but it should not be used mechanically for CPU-bound work. For heavy CPU-bound work, prefer:

- background workers;
- `Parallel.ForEach` or structured parallelism where appropriate;
- dedicated processing pipelines;
- batching and queueing work instead of creating too many tiny tasks.

The hot path should not create a new task per item when a small bounded scheduler or worker pool is sufficient.

## Keep APIs fast by default

Prefer low-allocation APIs in public boundaries:

- use `ReadOnlySpan<char>` for parser entry points;
- avoid returning `new string` when a slice or borrowed segment is enough;
- prefer immutable value types for domain data when their use is natural;
- avoid `ToList()` or `Select()` when a single pass loop is simpler and cheaper.

## Performance review checklist

Before closing a performance PR, ask:

- Was the issue reproduced and measured?
- Are there unnecessary allocations at the hot path?
- Is the code still readable, testable, and correct?
- Are there obvious pooling or span opportunities?
- Did the change improve the target metric without harming correctness?

## Related notes

- [[standards/csharp/csharp-coding-standard]]
- [[standards/csharp/csharp-modern-language-practices]]
- [[standards/concurrency/csharp-asynchronous-programming-practices]]
- [[standards/concurrency/concurrent-programming-practices]]
