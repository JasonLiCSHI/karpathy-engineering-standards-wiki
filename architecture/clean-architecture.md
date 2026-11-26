---
title: Clean Architecture
category: concepts
tags: [architecture, design, clean-architecture, testability, visibility/public]
aliases: [clean arch]
sources:
  - https://github.com/leewaiho/Clean-Architecture-zh
created: 2026-07-14T15:52:02+08:00
updated: 2026-07-31T09:58:28+08:00
summary: Language-agnostic rules for organizing use cases, business policies, components, boundaries, adapters, frameworks, composition, and tests so change remains local.
provenance:
  extracted: 0.78
  inferred: 0.22
  ambiguous: 0.00
base_confidence: 0.91
lifecycle: verified
lifecycle_changed: 2026-07-30
tier: core
---

# Clean Architecture

> Sources: [Clean Architecture Chinese translation](https://github.com/leewaiho/Clean-Architecture-zh) and Robert C. Martin's *Clean Architecture*. The rules below are synthesized and adapted rather than copied.

Clean Architecture is language- and framework-independent. Its purpose is to preserve both software behavior and the ability to change that behavior at a cost proportional to the change itself.

## Architectural objective

- Architecture must support use cases, operation, development, maintenance, deployment, and testing.
- Organize the system so business policy remains stable while delivery mechanisms, storage, frameworks, devices, and vendors can change locally.
- Keep important but non-urgent structural work visible; repeatedly trading architecture for urgent features raises the cost of every later change.
- Measure an architecture by how safely it permits change, not by the number of layers, projects, interfaces, or patterns it contains.
- Delay irreversible technology decisions until the system has enough evidence to make them responsibly.

## Dependency rule

Source-code dependencies point toward higher-level, more stable policy:

- Domain policy does not know about application orchestration, UI, databases, transport, frameworks, or composition.
- Application use cases may depend on domain policy and on abstractions required to complete a use case.
- Interface adapters translate between external representations and application-owned models.
- Infrastructure implements application-owned ports and contains storage, transport, framework, and device details.
- The composition root knows concrete types, creates the object graph, supplies configuration, and transfers control to the application.
- Runtime control flow may cross outward through an interface, but the compile-time dependency still points inward toward the interface owner.
- Concurrency mechanisms remain outer policy unless concurrency is itself part of the domain; follow [[standards/concurrency/concurrent-programming-practices]] for ownership, lifecycle, liveness, and execution-policy boundaries.

The rule concerns source dependencies, not data-flow direction. Stable platform primitives may be used directly; volatile details developed or selected by the team deserve explicit boundaries.

## Policies and layers

Layers are policy levels, not mandatory folder names:

| Policy level | Responsibility | Typical contents |
|---|---|---|
| Enterprise/domain policy | Rules that remain valid across application workflows | Entities, value objects, invariants, domain services |
| Application policy | Rules for achieving one user goal | Use cases, interactors, commands, queries, application ports |
| Interface adaptation | Translation across a boundary | Presenters, controllers, gateways, mappers, transport DTO conversion |
| Frameworks and drivers | Replaceable mechanisms close to input/output | UI frameworks, persistence, messaging, files, devices, web hosts |
| Composition | Select and connect concrete mechanisms | Main, dependency-injection registration, startup configuration |

- A higher-level policy is farther from input/output details and normally changes for more important, less frequent reasons.
- Group policies that change for the same actor, reason, and cadence; separate policies with different change axes.
- A physical deployment unit may contain several logical boundaries. Conversely, a justified boundary may require separate assemblies or services.
- Crossing a boundary has cost; introduce one only when it protects a meaningful policy or source of change.

## Boundary crossing

- Define an input model for what a use case needs, not a framework request object or persistence entity.
- Define an output model for what the caller may observe, not an internal domain graph.
- Pass simple, boundary-owned data structures across boundaries.
- Do not make inner models implement outer framework interfaces or inherit framework base classes.
- Place gateway or output-port interfaces on the policy side that needs them; outer adapters implement them.
- Translate exceptions and external status codes at the boundary that understands both meanings.
- Keep serialization, database schema, transport shape, and UI formatting outside domain and use-case policy.
- When partial failure is possible, the application policy defines atomicity, compensation, retry, and observable failure semantics.

## Use-case-centered structure

- Top-level names and modules should reveal the business capabilities and use cases before they reveal frameworks.
- Use [[architecture/domain-driven-design-practices]] when domain discovery, ubiquitous language, subdomains, bounded contexts, and aggregate consistency drive those capabilities.
- Each use case has an explicit input, observable result, authorization/validation policy, failure contract, and transaction boundary.
- Controllers receive and translate input; they do not own business decisions.
- Presenters format application output into a view model; views render that model with minimal logic.
- Repositories and gateways expose operations required by use cases, not generic access to a storage technology.
- A service boundary is not automatically an architectural boundary. A service with mixed policy and infrastructure remains coupled even when deployed separately.

## SOLID at architectural scale

- **SRP:** a module or component serves one actor or change axis, not merely "one function."
- **OCP:** protect established policy from confirmed variants through stable extension points; do not predict every future change.
- **LSP:** every implementation must preserve the promises, invariants, and failure semantics of its abstraction.
- **ISP:** callers depend only on the capability they use; avoid broad facades that couple unrelated use cases.
- **DIP:** high-level policy owns stable abstractions; volatile mechanisms depend on and implement them.

These principles complement [[standards/conventions/clean-code-practices]] but apply to modules and dependency boundaries rather than only functions and classes.

## Component cohesion

Use these forces together rather than treating one as absolute:

- **Reuse/Release Equivalence:** reusable code needs a deliberate versioned release boundary.
- **Common Closure:** place code that changes for the same reason and at the same time in one component.
- **Common Reuse:** do not force consumers to depend on code they do not use.

The tension is intentional. Stronger reuse may enlarge a component; stronger closure may group change-related code; stronger consumer isolation may split it. Choose according to the current dominant risk.

## Component coupling

- Keep the component dependency graph acyclic. Break cycles by moving shared policy, inverting a dependency, or separating a stable abstraction.
- Stable components should not depend on components that change more frequently or for less important reasons.
- A highly stable component should contain policy or useful abstractions; stability without abstraction can make concrete details difficult to replace.
- Dependency direction follows policy importance and stability, not runtime call order or organizational rank.
- Treat coupling metrics as diagnostic signals, not architecture targets.

## Frameworks, storage, and delivery

- Frameworks are tools and implementation details, not the architecture.
- Keep framework attributes, base classes, service location, and lifecycle callbacks outside core policy where practical.
- Use adapters or proxies when a framework requires inheritance or framework-owned interfaces.
- Database and ORM models belong to persistence adapters; map them to application or domain models at the boundary.
- Web, desktop, command line, RPC, and messaging are delivery choices. Core use cases should not require one delivery mechanism.
- A necessary framework dependency is still an explicit long-term trade-off and should be isolated as far outward as practical.

## Composition root

- Centralize concrete object creation, dependency-injection registration, configuration loading, and environment selection.
- Business objects receive dependencies normally and do not call the container.
- The composition root may depend on both policy abstractions and concrete adapters; inner components never depend on it.
- Different hosts, environments, or customers may use different composition roots over the same use cases.
- Keep startup decisions out of reusable contracts.

## Testability as an architecture property

- Tests are outer components: production code never depends on them, while tests depend inward on observable policy.
- Domain and use-case tests should run without a real UI, web server, database, filesystem, network, or dependency-injection framework.
- Separate hard-to-test mechanisms from testable decisions. Keep views, drivers, listeners, and persistence adapters deliberately simple.
- Test use-case contracts rather than mirroring every production class and private method.
- Supply fakes, stubs, or mocks through the same ports used by production adapters.
- A special test API is justified only when it reduces structural coupling and cannot leak unsafe capabilities into production.

## Design and delivery practice

1. Start from actors, use cases, business rules, invariants, and observable failures.
2. Identify independent change axes and volatile external details.
3. Draw only the boundaries needed to keep those changes local.
4. Assign ownership of models and interfaces to the policy side.
5. Define boundary data and error semantics before selecting adapters.
6. Implement the use case against replaceable ports and test it without infrastructure.
7. Add outer adapters and wire them in the composition root.
8. Verify dependency direction, component cycles, architecture tests, and end-to-end behavior.
9. Revisit boundaries when change history disproves the original assumptions.

For non-trivial changes, record these decisions using [[architecture/pre-development-design-document-standard]].

## Architecture smells

| Smell | Architectural response |
|---|---|
| Top-level structure names frameworks or technical layers only | Reorganize around business capabilities and use cases. |
| Domain references UI, persistence, transport, or DI packages | Move the dependency outward and introduce a policy-owned port if needed. |
| One facade serves unrelated actors | Split it into capability-focused use-case interfaces. |
| Generic repository exposes storage-shaped CRUD everywhere | Define operations in the language and transaction needs of use cases. |
| Framework DTOs travel through the whole system | Translate to boundary-owned input/output models. |
| Business tests require real infrastructure | Isolate the mechanism behind a port and test the policy directly. |
| Small adapter contains business branching | Move the decision into domain or application policy. |
| Shared project becomes a dumping ground | Split by common closure and common reuse. |
| Component cycle | Move shared policy, invert one dependency, or merge only if the change axis is truly shared. |
| Every feature requires edits across many components | Re-evaluate responsibility ownership and boundary placement. |
| Many interfaces have one stable implementation and no testing need | Remove speculative indirection unless it protects an actual boundary. |

## Practical limits

- Clean Architecture does not require four projects, concentric folders, entities for every noun, or interfaces for every class.
- A monolith can have clean boundaries; microservices can have tangled dependencies.
- Independent deployment is a business and operational decision, not proof of architectural quality.
- Do not isolate stable language or platform primitives merely to satisfy DIP mechanically.
- Do not add a boundary whose ongoing translation and coordination cost exceeds the change risk it protects.
- Performance, safety, regulatory, or hardware constraints may justify tighter coupling; document and test the trade-off.
- "Future flexibility" is not evidence. Prefer the simplest boundary that protects a demonstrated use case or volatility.

## Architecture review checklist

- [ ] Top-level structure communicates the system's business capabilities.
- [ ] Domain and application policy are independent of frameworks and delivery mechanisms.
- [ ] Source dependencies point toward higher-level, more stable policy.
- [ ] Each component has a clear actor, change axis, and release responsibility.
- [ ] The component graph is acyclic.
- [ ] Boundary interfaces and data are owned by the policy that needs them.
- [ ] Framework, storage, transport, and UI models do not leak inward.
- [ ] Use cases define validation, authorization, transaction, failure, and compensation semantics.
- [ ] Concrete creation and configuration remain in the composition root.
- [ ] Business rules can be tested without external infrastructure.
- [ ] Every abstraction and boundary protects a real responsibility or variation point.
- [ ] Deferred decisions remain replaceable rather than merely undocumented.

## Related

- [[standards/conventions/clean-code-practices]]
- [[architecture/domain-driven-design-practices]]
- [[standards/concurrency/concurrent-programming-practices]]
- [[architecture/pre-development-design-document-standard]]
- [[architecture/configuration-driven-layout]]
