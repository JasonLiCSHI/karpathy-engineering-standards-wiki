---
title: Domain-Driven Design Practices
category: skills
tags: [domain-modeling, strategic-design, tactical-design, architecture, visibility/public]
sources:
  - https://domain-driven-design.org/zh/ddd-concept-reference.html
  - https://ddd.academy/blog/
  - https://ddd.academy/blog/what-is-ddd-by-eric-evans
  - https://ddd.academy/blog/your-learning-journey
  - https://ddd.academy/blog/when-ddd-meets-integration
summary: Language-agnostic practices for discovering domains, evolving ubiquitous language, defining bounded contexts, designing aggregates, and integrating models without cargo-cult DDD.
provenance:
  extracted: 0.82
  inferred: 0.18
  ambiguous: 0.00
base_confidence: 0.92
lifecycle: verified
lifecycle_changed: 2026-07-30
tier: core
created: 2026-07-30T17:14:57+08:00
updated: 2026-07-30T17:14:57+08:00
---

# Domain-Driven Design Practices

> Sources: [DDD Concept Reference](https://domain-driven-design.org/zh/ddd-concept-reference.html) and selected [DDD Academy articles](https://ddd.academy/blog/). The guidance below synthesizes the sources into reusable practices rather than copying them.

Domain-Driven Design is a collaborative, model-driven approach for software whose main difficulty is domain complexity. It aligns business understanding, language, boundaries, software design, and code through continuous learning.

## Purpose and fit

- Use DDD to manage complex business rules, competing meanings, long-lived workflows, and knowledge distributed across domain and technical experts.
- Do not adopt DDD merely because a system has high traffic, many services, a database, or complex infrastructure. Those may be technical-complexity problems.
- Judge a model by whether it helps solve important domain problems, not by theoretical completeness or elegance.
- Prefer a deliberately simplified model suited to its purpose over a "realistic" model that attempts to represent everything.
- Concentrate modelling and engineering investment on the core domain that differentiates the organization.
- Buy, reuse, outsource, or simplify generic and supporting capabilities when custom modelling provides little strategic advantage.

## DDD is a continuous cycle

Treat DDD as three connected activities, not a one-way implementation process:

1. **Domain discovery:** learn the problem space with domain experts and expose rules, events, actors, friction, exceptions, and unknowns.
2. **Strategic design:** classify subdomains, define bounded contexts, assign ownership, and design relationships between models and teams.
3. **Tactical design:** implement one bounded context with entities, value objects, aggregates, repositories, domain events, policies, and services where they fit.
4. Return to discovery whenever scenarios, production evidence, or changed language invalidate the current model.

A code model that never feeds new questions back to domain discovery has stopped being domain-driven.

## Collaborative domain discovery

- Include people who can explain current behavior and people authorized to decide target behavior.
- Distinguish current-state (`As-Is`) facts from target-state (`To-Be`) decisions.
- Use concrete business scenarios before abstract diagrams.
- EventStorming can expose commands, actors, policies, external systems, domain events, hotspots, and process boundaries.
- Domain Storytelling can expose who performs work, with which artifacts, in what sequence, and where language differs.
- Keep sessions focused and psychologically safe; deliberately explore several imperfect alternatives instead of defending the first plausible model.
- Record decisions, terms, unresolved questions, examples, and rejected alternatives that future work must understand. Do not preserve every workshop artifact as permanent documentation.

## Ubiquitous language

- Build one shared language between domain experts and developers inside each bounded context.
- Put domain terms in conversations, scenarios, design documents, tests, APIs, events, and code.
- Define a term through examples, invariants, allowed transitions, and counterexamples rather than a dictionary sentence alone.
- Rename code and documentation when domain understanding changes.
- Treat recurring translation between business language and code language as a model defect.
- Do not force one term or model to mean the same thing across the whole enterprise.
- When the same word has different rules, lifecycle, identity, or owners, make the context explicit or use different names.

## Problem space and subdomains

- Start by identifying the business problem, actors, outcomes, policies, constraints, and sources of uncertainty.
- Decompose the domain into subdomains to reduce cognitive load.
- Classify each subdomain:
  - **Core:** creates strategic differentiation and deserves the strongest modelling and most experienced attention.
  - **Supporting:** necessary for the core business but not itself differentiating; keep it fit for purpose.
  - **Generic:** broadly solved elsewhere; prefer established products or standard capabilities.
- Revisit classification as business strategy changes.
- Do not infer software or service boundaries directly from organization charts, database schemas, or current applications.

## Purpose-driven model exploration

- Generate multiple competing models before selecting one for an important domain problem.
- Evaluate each model against real scenarios, especially exceptions, mid-process changes, failure recovery, reporting, and future decisions already visible in the business.
- State what each model emphasizes, what it deliberately ignores, and which scenario makes it awkward.
- Choose the model that makes the important decisions and invariants easiest to express.
- A new problem may justify evolving the existing model or introducing a separate bounded context; do not automatically stretch one model across incompatible purposes.
- Prefer model fit over model perfection. A technically elegant abstraction that domain experts cannot recognize is not a successful domain model.

## Bounded contexts

A bounded context defines where one model and ubiquitous language are internally consistent.

- Give every bounded context an explicit business purpose, language, model, team owner, and integration surface.
- Keep entities, value objects, aggregates, repositories, commands, and events inside the context that defines their meaning.
- Do not share one domain model or database entity across contexts merely to remove duplicated fields.
- Cross a context boundary through explicit contracts and translation.
- Duplicate small amounts of data or representation when that protects model autonomy.
- A bounded context is a semantic boundary, not automatically a process, service, repository, namespace, team, or deployment unit.
- Split a context when one term carries incompatible meanings or independent models must evolve for different business reasons.
- Merge only when separate models and translation add cost without protecting a real difference.

## Context mapping and integration

- Maintain a context map that shows upstream/downstream influence, ownership, contracts, and translation responsibilities.
- Decide integration from business flow and quality attributes, not from fashion or a default transport.
- Treat interoperability as a strategic design concern: the spaces between systems carry business semantics, failure modes, ownership, and organizational politics.
- Use an anti-corruption layer when an external or legacy model would otherwise distort the local model.
- Publish contracts that are stable for consumers; keep internal aggregates and persistence models private.
- Make delivery guarantees, ordering, idempotency, retries, timeouts, compatibility, and partial failure explicit.
- Design observability, security, scalability, and testability as integration requirements, not post-deployment additions.
- Avoid uncontrolled point-to-point integrations; record why an API, event, broker, batch, or shared platform fits the specific business interaction.

## Tactical model building blocks

Use tactical patterns only when they express the discovered model:

### Entities

- Use identity when the business tracks continuity across state changes.
- Keep identity stable and independent of mutable descriptive attributes.
- Put behavior that protects entity rules with the entity or its aggregate.

### Value objects

- Model descriptive concepts by their complete value rather than identity.
- Make them immutable where practical and validate them at creation.
- Use value objects for meaningful units, ranges, identifiers, money, addresses, and other concepts where primitives permit invalid combinations.

### Aggregates and aggregate roots

- Define an aggregate as the smallest consistency boundary that must enforce invariants atomically.
- Expose mutation through one aggregate root.
- Do not allow callers to modify aggregate internals behind the root.
- Reference other aggregates by identity unless a scenario requires a separately loaded representation.
- Keep transactions local to one aggregate where practical; coordinate multi-aggregate workflows in the application layer and accept explicit eventual consistency when appropriate.
- Keep aggregates small enough for clear ownership and concurrency, but large enough to enforce the invariants that truly require immediate consistency.

### Commands, policies, and domain events

- A command expresses an actor's intent and may be rejected.
- A policy or rule decides what may happen under stated conditions.
- A domain event records a business fact that has happened and matters to domain experts.
- Name commands as imperatives and events in past tense.
- Keep events immutable and include the domain identity and facts consumers need; do not expose an internal object graph.
- Do not use events merely to avoid direct calls inside one cohesive aggregate.

### Repositories and factories

- Define repositories around aggregate roots and domain-oriented retrieval needs.
- Make a repository resemble a collection of aggregates, not a generic table gateway.
- Keep persistence queries, ORM types, and database transactions in infrastructure implementations.
- Use a factory when valid creation is complex, selects among meaningful variants, or requires knowledge that does not belong to the created object.
- Prefer a constructor or named creation method when creation is simple.

### Domain services, application services, and specifications

- Put behavior on an entity or value object when it naturally belongs there.
- Use a domain service for a stateless domain operation that spans concepts but belongs to the domain language.
- Use an application service to authorize, orchestrate a use case, load aggregates, invoke domain behavior, coordinate transactions, and publish results.
- Use infrastructure services for technical mechanisms such as storage, transport, clocks, or external providers.
- Use a specification when a named, composable business predicate improves clarity or reuse; do not wrap every conditional in a specification.

## Layer and dependency guidance

- Interface adapters translate protocols and representations.
- Application code owns use-case sequencing, actor/permission checks, transaction coordination, and consistency across boundaries.
- Domain code owns business meaning, invariants, and valid state transitions.
- Infrastructure code implements persistence and external-system adapters.
- Dependencies point toward domain and application policy as described by [[architecture/clean-architecture]].
- Logical boundaries do not dictate deployment. A modular monolith may preserve contexts better than prematurely distributed services.

## Testing the model

- Express scenarios using ubiquitous language.
- Test aggregate invariants, valid transitions, rejected commands, value semantics, and emitted domain events.
- Test application orchestration separately from domain policy.
- Use contract tests at context and external-system boundaries.
- Test translation and anti-corruption behavior with examples from both models.
- Do not couple tests to private object structure or persistence layout.
- Add scenario-based tests before refactoring a model so alternative designs can be compared against the same business evidence.

## Design-document practice

For domain-complex changes, the design document should include:

- actors, goals, representative scenarios, exceptions, and unresolved domain questions;
- ubiquitous-language definitions and terms that differ by context;
- subdomain classification and why the work belongs to the core, supporting, or generic domain;
- bounded-context ownership and a context map for every changed interaction;
- aggregate boundaries, roots, invariants, commands, events, policies, and consistency requirements;
- alternatives considered and scenarios used to select the model;
- integration semantics, translation ownership, and operational failure behavior;
- evidence and tests that will show whether the model remains fit.

See [[architecture/pre-development-design-document-standard]] for the full implementation gate.

## Common DDD smells

## Practical limits

- DDD is not required for simple CRUD or primarily technical-complexity systems.
- Strategic design can be valuable without adopting every tactical pattern.
- Tactical patterns without discovery and bounded contexts are object modelling, not full DDD.
- Do not create a domain service, repository, factory, specification, or event unless it expresses a real domain concept or boundary.
- Eventual consistency is a business trade-off, not a default badge of sophistication.
- Do not distribute a system merely because bounded contexts exist.
- A model is temporary knowledge. Change it when business evidence changes, while preserving explicit compatibility where required.

## Review checklist

- [ ] The design starts from an important domain problem and concrete scenarios.
- [ ] Domain and technical complexity are distinguished.
- [ ] Core, supporting, and generic subdomains have deliberate investment strategies.
- [ ] Ubiquitous language is defined within explicit bounded contexts.
- [ ] Multiple meaningful model alternatives were considered for high-impact decisions.
- [ ] The selected model makes important rules and exceptions explicit.
- [ ] Aggregate boundaries match immediate consistency requirements.
- [ ] Commands, policies, events, repositories, factories, and services have distinct responsibilities.
- [ ] Context relationships, contracts, and translation ownership are explicit.
- [ ] Integration failure, compatibility, observability, security, and idempotency are designed.
- [ ] Domain behavior can be tested without infrastructure.
- [ ] Logical model boundaries are not confused with deployment topology.

## Related

- [[architecture/clean-architecture]]
- [[standards/conventions/clean-code-practices]]
- [[architecture/pre-development-design-document-standard]]
