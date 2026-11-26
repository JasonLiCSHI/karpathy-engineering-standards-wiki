# Karpathy Engineering Standards Wiki

A reusable, public-safe **Obsidian + AI Coding Agent engineering knowledge base** (28 distilled pages covering Clean Architecture, DDD, Clean Code, Modern C#, Async/Concurrency, Performance, Logging, Naming, Error Codes, and AI Wiki Governance).

> **Skill Attribution & Direct Reuse**
>
> This repository is a curated, sanitized **knowledge output accumulated using the `llm-wiki` skill** (inspired by Andrej Karpathy's [LLM Wiki](https://x.com/karpathy) architecture) and designed to work directly alongside the **`karpathy-guidelines` skill** from [`multica-ai/andrej-karpathy-skills`](https://github.com/multica-ai/andrej-karpathy-skills).
>
> Every page is vendor-neutral, stripped of proprietary project details, and ready to clone and reuse directly in your own Obsidian vault or AI coding agent workflow.

## Why This Exists

- **Accumulated via `llm-wiki`**: Instead of re-prompting engineering standards from scratch every session, recurring architectural principles, language guidelines, concurrency pitfalls, and review checklists are distilled into compact, cross-linked Markdown pages with YAML frontmatter (`summary`, `tier`, `relationships`, `visibility/public`).
- **Paired with `karpathy-guidelines`**: While [`multica-ai/andrej-karpathy-skills`](https://github.com/multica-ai/andrej-karpathy-skills) governs *how* an LLM codes (*Think Before Coding*, *Simplicity First*, *Surgical Changes*, *Goal-Driven Execution*), this wiki supplies the concrete **engineering standards** (*what* good C#, architecture, concurrency, performance, logging, naming, and design documents look like).

## Quick Start (Direct Reuse)

### 1. Use as an Obsidian Vault

```bash
git clone https://github.com/JasonLiCSHI/karpathy-engineering-standards-wiki.git
```

Open the cloned folder as a vault in **Obsidian** (start from [index.md](index.md) or [hot.md](hot.md)). All internal links use Obsidian wikilinks and work seamlessly in Graph View.

### 2. Connect to Your AI Coding Agent (`llm-wiki` + `karpathy-guidelines`)

1. Point your `llm-wiki` / `wiki-query` config (`OBSIDIAN_VAULT_PATH`) at the cloned folder so your agent reads `hot.md` → `index.md` → targeted pages via progressive loading.
2. Install the companion `karpathy-guidelines` skill from [`multica-ai/andrej-karpathy-skills`](https://github.com/multica-ai/andrej-karpathy-skills) (`skills/karpathy-guidelines/SKILL.md`) into your agent's skills directory (`.copilot/skills/`, `.claude/skills/`, or `.agents/skills/`).
3. Optional: index the vault with `qmd` for fast local semantic + BM25 retrieval.

## Knowledge Catalog (28 Pages)

### Governance & Wiki Methodology

- [Progressive Knowledge Loading](governance/progressive-loading.md) — Defines the L0, L1, and L2+ reading depths used to retrieve wiki knowledge efficiently without confusing them with the folder hierarchy.
- [Wiki Retrieval, Conflict Resolution, and Update Rules](governance/wiki-governance-rules.md) — Operational rules for progressive wiki retrieval, source-of-truth conflict resolution, and anti-drift maintenance in an AI-assisted Obsidian Wiki.

### Architecture & Design

- [Clean Architecture](architecture/clean-architecture.md) — Language-agnostic rules for organizing use cases, business policies, components, boundaries, adapters, frameworks, composition, and tests so change remains local.
- [Configuration-Driven Layout](architecture/configuration-driven-layout.md) — Configuration-driven layout stores UI-independent content placement in configuration while leaving rendering details to the page.
- [Domain-Driven Design Practices](architecture/domain-driven-design-practices.md) — Language-agnostic practices for discovering domains, evolving ubiquitous language, defining bounded contexts, designing aggregates, and integrating models without cargo-cult DDD.
- [Pre-Development Design Document Standard](architecture/pre-development-design-document-standard.md) — A development gate and minimum design-document template for changes with domain, contract, data-flow, configuration, persistence, or compatibility impact.

### Clean Code & Naming

- [Clean Code Practices](standards/conventions/clean-code-practices.md) — Practical rules for readable names, focused functions, cohesive modules, explicit dependencies, safe errors, useful comments, continuous refactoring, and clean-code review.
- [Naming Standard](standards/naming/naming-standard.md) — Repository-wide naming standard for code, APIs, logs, and business identifiers.
- [Backend naming standard](standards/naming/backend-naming-standard.md) — Naming rules for service and backend code.
- [Frontend naming standard](standards/naming/frontend-naming-standard.md) — Naming rules for frontend code.

### Modern C#

- [Modern C# Programming Standard](standards/csharp/modern-csharp-standard.md) — Modern C# programming standard for the repository.
- [C# Coding Standard](standards/csharp/csharp-coding-standard.md) — Enforceable C# and clean-code rules covering language use, design clarity, naming, exceptions, async code, LINQ, ReSharper cleanupcode/inspectcode with shared .DotSettings, and reviews.
- [C# Language-Version-Aware Coding](standards/csharp/csharp-language-version-aware-coding.md) — C# projects should actively evaluate useful features available in their effective language version, while adopting them only when they improve semantics, safety, or clarity.
- [C# Modern Language Practices](standards/csharp/csharp-modern-language-practices.md) — A practical repo standard for .NET 10 / C# 14-era coding: use modern features when they improve safety, clarity, and correctness, not merely because they are new.

### Concurrency & Async

- [Async I/O and Cancellation](standards/concurrency/async-io-and-cancellation.md) — Standard for async I/O and cancellation in C# service code.
- [C# Asynchronous Programming Practices](standards/concurrency/csharp-asynchronous-programming-practices.md) — Enforceable C# async rules for TAP APIs, workload selection, task composition, cancellation, exceptions, context, streams, resource lifetime, testing, and review.
- [Concurrent Programming Practices](standards/concurrency/concurrent-programming-practices.md) — Language-agnostic rules for shared-state safety, task execution, cancellation, liveness, scalability, memory visibility, synchronization, and concurrent testing.
- [Thread Safety and Parallelism](standards/concurrency/thread-safety-and-parallelism.md) — Standard for thread safety and parallelism in C# code.

### Performance & Low Allocation

- [C# Performance Practices](standards/performance/csharp-performance-practices.md) — Performance code in modern .NET should be explicit, well-measured, and low-allocation by default when the hot path justifies it.
- [Benchmarking and Profiling](standards/performance/benchmarking-and-profiling.md) — Standard for benchmarking and profiling hot C# paths.
- [Low-Allocation and Span Patterns](standards/performance/low-allocation-and-span.md) — Low-allocation patterns using Span<T>, Memory<T>, and ArrayPool.

### Logging & Observability

- [Logging Standard](standards/logging/logging-standard.md) — Repository-wide standard for application and service logging.
- [Backend Logging Standard](standards/logging/backend-logging-standard.md) — Service-side application and infrastructure logging rules.
- [Frontend Logging Standard](standards/logging/frontend-logging-standard.md) — Frontend logging rules.
- [Operations and Alerting Standard](standards/logging/operations-and-alerting-standard.md) — Operations and alerting standard.

### Error Codes

- [Error Code Standard](standards/error-codes/error-code-standard.md) — Repository-wide error-code taxonomy and handling standard.
- [Backend error codes](standards/error-codes/backend-error-codes.md) — Backend error-code structure and conventions.
- [Frontend error codes](standards/error-codes/frontend-error-codes.md) — Frontend error-code structure and conventions.

## License

MIT — see [LICENSE](LICENSE). Companion guidelines credited to [`multica-ai/andrej-karpathy-skills`](https://github.com/multica-ai/andrej-karpathy-skills) (MIT).
