---
title: Hot Cache
updated: 2026-11-26T14:00:00+08:00
---

# Hot Cache

*Compact semantic snapshot for fast AI agent onboarding.*

## Recent Activity

- [2026-11-26T14:00:00+08:00] PUBLIC_EXPORT — Published initial 28-page public engineering standards wiki accumulated via `llm-wiki` and paired with `karpathy-guidelines` (`multica-ai/andrej-karpathy-skills`).

## Active Threads

- [[architecture/clean-architecture]] & [[architecture/domain-driven-design-practices]] — Dependency direction, aggregate boundaries, and pre-development design gates.
- [[standards/csharp/csharp-coding-standard]] & [[standards/csharp/modern-csharp-standard]] — Practical C# style, language-version-aware adoption, and modern idioms.
- [[standards/concurrency/csharp-asynchronous-programming-practices]] & [[standards/concurrency/concurrent-programming-practices]] — True async I/O, cancellation propagation, thread safety, and deadlock avoidance.
- [[standards/performance/csharp-performance-practices]] & [[standards/performance/low-allocation-and-span]] — Measurement-first optimization, `Span<T>`, and low-allocation hot paths.
- [[governance/progressive-loading]] & [[governance/wiki-governance-rules]] — Low-token progressive wiki retrieval and anti-drift maintenance rules.

## Key Invariants

- **Pair behavioral discipline with concrete standards**: Apply `karpathy-guidelines` (*Think Before Coding*, *Simplicity First*, *Surgical Changes*, *Goal-Driven Execution*) together with the domain-specific standards in `index.md`.
- **Progressive loading first**: Read `hot.md` → `index.md` → only the top matching pages (`tier: core` before `tier: supporting`).
