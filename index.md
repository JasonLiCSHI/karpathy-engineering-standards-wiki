---
title: Public Engineering Standards Wiki Index
updated: 2026-11-26T14:00:00+08:00
---

# Public Engineering Standards Wiki Index

This index lists all 28 reusable engineering standards distilled with the `llm-wiki` pattern and designed to pair with `karpathy-guidelines`.

## Governance & Wiki Methodology

- [[governance/progressive-loading|Progressive Knowledge Loading]] — Defines the L0, L1, and L2+ reading depths used to retrieve wiki knowledge efficiently without confusing them with the folder hierarchy.
- [[governance/wiki-governance-rules|Wiki Retrieval, Conflict Resolution, and Update Rules]] — Operational rules for progressive wiki retrieval, source-of-truth conflict resolution, and anti-drift maintenance in an AI-assisted Obsidian Wiki.

## Architecture & Design

- [[architecture/clean-architecture|Clean Architecture]] — Language-agnostic rules for organizing use cases, business policies, components, boundaries, adapters, frameworks, composition, and tests so change remains local.
- [[architecture/configuration-driven-layout|Configuration-Driven Layout]] — Configuration-driven layout stores UI-independent content placement in configuration while leaving rendering details to the page.
- [[architecture/domain-driven-design-practices|Domain-Driven Design Practices]] — Language-agnostic practices for discovering domains, evolving ubiquitous language, defining bounded contexts, designing aggregates, and integrating models without cargo-cult DDD.
- [[architecture/pre-development-design-document-standard|Pre-Development Design Document Standard]] — A development gate and minimum design-document template for changes with domain, contract, data-flow, configuration, persistence, or compatibility impact.

## Clean Code & Naming

- [[standards/conventions/clean-code-practices|Clean Code Practices]] — Practical rules for readable names, focused functions, cohesive modules, explicit dependencies, safe errors, useful comments, continuous refactoring, and clean-code review.
- [[standards/naming/naming-standard|Naming Standard]] — Repository-wide naming standard for code, APIs, logs, and business identifiers.
- [[standards/naming/backend-naming-standard|Backend naming standard]] — Naming rules for service and backend code.
- [[standards/naming/frontend-naming-standard|Frontend naming standard]] — Naming rules for frontend code.

## Modern C#

- [[standards/csharp/modern-csharp-standard|Modern C# Programming Standard]] — Modern C# programming standard for the repository.
- [[standards/csharp/csharp-coding-standard|C# Coding Standard]] — Enforceable C# and clean-code rules covering language use, design clarity, naming, exceptions, async code, LINQ, ReSharper cleanupcode/inspectcode with shared .DotSettings, and reviews.
- [[standards/csharp/csharp-language-version-aware-coding|C# Language-Version-Aware Coding]] — C# projects should actively evaluate useful features available in their effective language version, while adopting them only when they improve semantics, safety, or clarity.
- [[standards/csharp/csharp-modern-language-practices|C# Modern Language Practices]] — A practical repo standard for .NET 10 / C# 14-era coding: use modern features when they improve safety, clarity, and correctness, not merely because they are new.

## Concurrency & Async

- [[standards/concurrency/async-io-and-cancellation|Async I/O and Cancellation]] — Standard for async I/O and cancellation in C# service code.
- [[standards/concurrency/csharp-asynchronous-programming-practices|C# Asynchronous Programming Practices]] — Enforceable C# async rules for TAP APIs, workload selection, task composition, cancellation, exceptions, context, streams, resource lifetime, testing, and review.
- [[standards/concurrency/concurrent-programming-practices|Concurrent Programming Practices]] — Language-agnostic rules for shared-state safety, task execution, cancellation, liveness, scalability, memory visibility, synchronization, and concurrent testing.
- [[standards/concurrency/thread-safety-and-parallelism|Thread Safety and Parallelism]] — Standard for thread safety and parallelism in C# code.

## Performance & Low Allocation

- [[standards/performance/csharp-performance-practices|C# Performance Practices]] — Performance code in modern .NET should be explicit, well-measured, and low-allocation by default when the hot path justifies it.
- [[standards/performance/benchmarking-and-profiling|Benchmarking and Profiling]] — Standard for benchmarking and profiling hot C# paths.
- [[standards/performance/low-allocation-and-span|Low-Allocation and Span Patterns]] — Low-allocation patterns using Span<T>, Memory<T>, and ArrayPool.

## Logging & Observability

- [[standards/logging/logging-standard|Logging Standard]] — Repository-wide standard for application and service logging.
- [[standards/logging/backend-logging-standard|Backend Logging Standard]] — Service-side application and infrastructure logging rules.
- [[standards/logging/frontend-logging-standard|Frontend Logging Standard]] — Frontend logging rules.
- [[standards/logging/operations-and-alerting-standard|Operations and Alerting Standard]] — Operations and alerting standard.

## Error Codes

- [[standards/error-codes/error-code-standard|Error Code Standard]] — Repository-wide error-code taxonomy and handling standard.
- [[standards/error-codes/backend-error-codes|Backend error codes]] — Backend error-code structure and conventions.
- [[standards/error-codes/frontend-error-codes|Frontend error codes]] — Frontend error-code structure and conventions.
