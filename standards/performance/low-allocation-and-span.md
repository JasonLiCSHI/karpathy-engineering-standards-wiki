---
title: Low-Allocation and Span Patterns
category: concepts
tags: [csharp, performance, low-allocation, span, visibility/public]
sources: [distilled-engineering-standards]
summary: Low-allocation patterns using Span<T>, Memory<T>, and ArrayPool.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: supporting
relationships:
  - target: "[[standards/performance/csharp-performance-practices]]"
    type: extends
---

# Low-Allocation and Span Patterns

This section focuses on the most common patterns used to reduce allocation pressure and improve throughput in modern .NET code.

## 1. When to use `Span<T>`

Use `Span<T>` when you need to operate on a contiguous region of memory without creating extra copies.

Typical use cases:

- parsing text from a buffer
- slicing a byte array for protocol decoding
- processing incoming payloads incrementally
- searching or matching within a buffer without converting to a new string

```csharp
public static bool HasPrefix(ReadOnlySpan<char> text, ReadOnlySpan<char> prefix)
{
    return text.StartsWith(prefix, StringComparison.Ordinal);
}
```

## 2. Use `ReadOnlySpan<T>` for read-only data

If the data is not modified, prefer `ReadOnlySpan<T>`.

This helps signal intent and avoids accidental mutation in parser code.

## 3. Avoid string churn in tight loops

The most common performance bug is repeatedly creating strings during iteration.

Bad:

```csharp
foreach (var item in items)
{
    var key = item.Name.ToLowerInvariant();
    // repeated string creation and parsing
}
```

Better:

- parse once into a meaningful value
- reuse buffers when the algorithm requires it
- keep intermediate values local and finite

## 4. Prefer stackalloc for short-lived buffers

When the buffer is small and short-lived, `stackalloc` can avoid heap churn.

```csharp
Span<int> values = stackalloc int[16];
```

Use it only when:

- the lifetime is local and clear
- the size is small and bounded
- the code remains readable and safe

## 5. Use pooling only when the pattern is proven

`ArrayPool<T>` is useful for high-throughput workloads that repeatedly allocate large buffers. It should not be used by default for all code.

Pool only when:

- the buffer size is large enough to matter
- the allocation happens frequently
- the lifetime is explicit and manageable

## 6. Recommendation

For this repository, standardize on this rule:

- use `Span<T>`/`ReadOnlySpan<T>` for buffer and parsing logic when it improves data handling without making code opaque
- avoid string creation in hot loops
- use pooling only where allocation pressure is real and measurable
- keep the code understandable to the team
