---
title: Pre-Development Design Document Standard
category: skills
tags: [design, architecture, conventions, documentation, testing, visibility/public]
sources:
  - https://domain-driven-design.org/zh/ddd-concept-reference.html
  - https://ddd.academy/blog/
  - https://itmyhome.com/java-concurrent-programming-combat/java-concurrent-programming-combat.pdf
summary: A development gate and minimum design-document template for changes with domain, contract, data-flow, configuration, persistence, or compatibility impact.
provenance:
  extracted: 0.79
  inferred: 0.20
  ambiguous: 0.01
base_confidence: 0.87
lifecycle: draft
lifecycle_changed: 2026-07-22
tier: core
created: 2026-07-22T14:07:26+08:00
updated: 2026-07-31T09:58:28+08:00
---

# Pre-Development Design Document Standard

For non-trivial development, a design document is required before implementation begins. This applies whenever a change introduces or changes a domain model, public contract, module boundary, data flow, configuration or persistence shape, lifecycle, compatibility behavior, or cross-component interaction. Small isolated fixes may use a concise design note, but it must still state the problem, intended behavior, affected boundary, and verification.

## Required content

### 1. Context, goals, and scope

- Describe the business or technical problem, the current limitation, and the desired outcome.
- State measurable or observable success criteria.
- Explicitly list non-goals and responsibilities that remain outside the proposed module or feature.
- Identify affected users, callers, data, and existing behavior.

### 2. Current state, dependencies, and boundaries

- Explain the relevant existing architecture, modules, contracts, data structures, and dependencies.
- Define the proposed responsibility boundary: what the component owns, what it delegates, and what it must not do.
- Identify stable public contracts versus implementation-only types; cross-module consumers must not depend on implementation details.
- State dependency-injection and runtime-data ownership boundaries: replaceable collaborators are injected, while per-request/per-document runtime data is materialized explicitly.

### 3. Domain model and invariants

- For domain-complex changes, document representative business scenarios, ubiquitous language, subdomain classification, bounded contexts, and model alternatives before selecting tactical patterns.
- Define core concepts, their identities, ownership, relationships, and terminology.
- Describe required invariants, uniqueness rules, ordering, and state transitions.
- Specify which model is the source of truth and which models are derived projections.
- Call out invalid or prohibited states and the failure that results from each.
- Define aggregate roots and immediate consistency boundaries; distinguish commands, policies, and domain events.
- Identify context relationships, integration contracts, translation ownership, and any anti-corruption layer.

### 4. Architecture and interfaces

- Provide a module/component diagram or equivalent responsibility map for non-trivial work.
- Define public interfaces, inputs, outputs, events, commands, DTOs, and their ownership.
- Explain extension points and registration/discovery rules where behavior is plugin-based.
- Explain why the selected separation of responsibilities supports the goals and avoids unwanted coupling. ^[inferred]

### 5. Behavior, flows, and lifecycle

- Document the normal end-to-end flow with a sequence, state, or data-flow diagram when multiple components interact.
- Cover initialization/loading, mutation/update, notification/propagation, disposal/unloading, and rebuild/refresh behavior when applicable.
- Define ordering guarantees, cancellation points, transactions, rollback, retries, and idempotency as relevant.
- Record causality or traceability data required to diagnose a multi-step change.
- For concurrent behavior, define task decomposition, execution policy, shared-state ownership, atomicity, visibility, capacities, backpressure, cancellation, shutdown, liveness risks, and operational diagnostics.

### 6. Data, configuration, and persistence

- Define data formats, schemas, identifiers, validation rules, and compatibility with existing files or APIs.
- Separate declarative configuration from runtime behavior and presentation details; configuration should state stable content and logical placement, not implementation mechanics or UI rendering hints.
- Specify persistence ownership, read/write synchronization, integrity validation, migrations, and backward-compatible input handling.
- Show representative valid and invalid examples when the format is a key part of the design.

### 7. Errors, compatibility, and operational behavior

- Enumerate expected invalid inputs, configuration errors, missing dependencies, and business-rule failures.
- Define the explicit error/result contract; do not silently ignore invalid configuration or unrecoverable failures.
- State backward-compatibility commitments, migration steps, retained events/APIs, and behavior deliberately changed or removed.
- Describe diagnostics, logging, auditing, performance concerns, and security/integrity constraints where they affect the design.

### 8. Verification and evolution

- Define test scenarios that protect contracts, invariants, error paths, compatibility, lifecycle, and cross-component flows.
- Identify realistic test data and what it represents; use round-trip, integration, or architecture-guard tests when unit tests alone cannot prove the design.
- Record open decisions, risks, assumptions, rollout/migration stages, and the intended evolution path.
- Map each important requirement to a test, validation rule, or observable acceptance criterion.

## Completion gate

Implementation may start only when the document:

1. Makes the problem, goals, non-goals, and affected boundaries unambiguous.
2. Defines the model, public contract, ownership, and invariants needed for implementation.
3. Explains the end-to-end behavior and failure/rollback path for changes that cross components.
4. Specifies configuration/data compatibility and migration behavior when persisted or external data is affected.
5. Lists the verification strategy, including the tests that prove the intended behavior and protect regressions.
6. Records unresolved decisions as explicit open items rather than hiding them in implementation assumptions.
7. For domain-complex work, shows that the chosen model and boundaries were evaluated against concrete scenarios rather than selected from a pattern catalog.
8. For concurrent work, proves that safety, liveness, lifecycle, overload, and performance have explicit contracts and verification.

## Review checklist

- Can another engineer implement the change without rediscovering the domain model or boundary decisions?
- Would a domain expert recognize the language, rules, scenarios, and context-specific meanings used by the design?
- Does every concurrent state, task, queue, cancellation path, and shutdown responsibility have an explicit owner and bounded behavior?
- Is every public API, event, data format, and configuration field assigned an owner and a purpose?
- Are normal flow, failure flow, and lifecycle flow all described?
- Are backward compatibility and migration consequences explicit?
- Do tests validate observable behavior and architecture boundaries rather than only private implementation details?
- Is the document concise enough to guide delivery, while detailed enough to prevent ambiguous implementation? ^[inferred]

## Related

- [[architecture/clean-architecture]]
- [[architecture/domain-driven-design-practices]]
- [[standards/concurrency/concurrent-programming-practices]]
- [[architecture/configuration-driven-layout]]
