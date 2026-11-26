---
title: Benchmarking and Profiling
category: concepts
tags: [csharp, performance, profiling, visibility/public]
sources: [distilled-engineering-standards]
summary: Standard for benchmarking and profiling hot C# paths.
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

# Benchmarking and Profiling

Performance work in .NET must be measured. This section defines the repository expectation for benchmarking and profiling before and after optimization work.

## 1. Measure first

Before changing a hot path, establish a baseline.

Use measurements for:

- time per operation
- allocation count
- latency distribution
- throughput under load
- peak memory footprint

Do not optimize purely based on intuition.

## 2. Use the right tools

For .NET code, the standard tools include:

- BenchmarkDotNet for micro-benchmarks
- dotnet-trace for runtime sampling
- dotnet-counters for live metric inspection
- dotnet-gcdump for memory and GC analysis
- performance counters and app telemetry for production path analysis

## 3. Separate correctness from optimization

Before measuring performance, confirm correctness:

- inputs are validated
- edge cases are covered
- concurrency and cancellation are handled
- the code path is deterministic under test

An optimized but incorrect implementation is not acceptable.

## 4. Compare apples to apples

When benchmarking, keep the factors stable:

- same input size
- same environment
- same runtime version
- same build configuration
- same warm-up behavior

A benchmark with drift or hidden noise is not reliable.

## 5. Track allocations as part of the result

For modern .NET code, milliseconds alone are not enough.

Measure:

- total allocated bytes
- GC collection counts
- object churn in loops
- repeated parsing or copying

This is especially important for service pipelines and data / stream processing code.

## 6. Optimize the real bottleneck

A hot path may appear to be slow because of logging, serialization, or network latency rather than the local algorithm itself. Always validate where the time is actually being spent.

## 7. Recommended repo standard

The repository should require:

- benchmark or profile evidence before large optimization work
- clear comparison before and after performance changes
- explicit reasoning about allocations in hot paths
- trace data or counters when a fix affects throughput or latency

This keeps optimization work honest, repeatable, and maintainable.
