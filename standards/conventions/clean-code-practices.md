---
title: Clean Code Practices
category: skills
tags: [clean-code, refactoring, code-review, design, visibility/public]
sources:
  - https://www.freecodecamp.org/chinese/news/the-clean-code-handbook/
created: 2026-07-30T17:05:45+08:00
updated: 2026-11-20T15:25:00+08:00
summary: Practical rules for readable names, focused functions, cohesive modules, explicit dependencies, safe errors, useful comments, continuous refactoring, and clean-code review.
provenance:
  extracted: 0.76
  inferred: 0.24
  ambiguous: 0.00
base_confidence: 0.90
lifecycle: verified
lifecycle_changed: 2026-07-30
tier: core
relationships:
  - target: "[[architecture/domain-driven-design-practices]]"
    type: related_to
---

# Clean Code Practices

> Sources: [The Clean Code Handbook](https://www.freecodecamp.org/chinese/news/the-clean-code-handbook/) and Robert C. Martin's *Clean Code: A Handbook of Agile Software Craftsmanship*. The principles below are synthesized and adapted rather than copied.

These practices apply across programming languages and paradigms; language-specific standards add syntax and ecosystem rules without replacing them.

Clean code minimizes the effort required to understand, verify, change, and safely extend behavior.

## Reader-first rule

- Optimize for the next reader, not for the fewest characters typed.
- Code should reveal its intent, decision points, ownership, and failure paths without requiring the reader to simulate many hidden steps.
- Prefer obvious control flow over clever expressions.
- Keep related statements close and separate distinct concerns with structure rather than explanatory prose.
- Use repository-wide formatting tools so visual consistency is automatic.
- A change is incomplete when it works but leaves the affected behavior harder to understand than before.

## Names reveal intent

- Use domain terms that describe responsibility, state, unit, and lifecycle.
- A type name should identify one concept; do not reuse an established term for a lighter or unrelated object.
- Methods use verbs or verb phrases that state observable behavior.
- Boolean names should read as predicates, such as `IsValid`, `CanDelete`, or `HasRanges`.
- Collection names are plural and describe their elements.
- Include units or boundaries when ambiguity is possible, such as `timeoutMilliseconds` or `maximumRangeCount`.
- Avoid abbreviations, type-encoded names, vague words such as `data`, `info`, `manager`, or `helper`, and distinctions based only on numbers.
- Rename code when its current name requires a comment to explain what it really means.

## Focused functions

- A function should operate at one responsibility and one level of abstraction.
- Separate orchestration from detail: a use-case method should describe the flow, while focused collaborators implement persistence, mapping, validation, or transport details.
- Keep parameters few and cohesive. Introduce a value object when several parameters always travel together and form one concept.
- Avoid boolean parameters that make calls read ambiguously; use separate named operations or an explicit option type when the modes have different meaning.
- Make side effects visible in the method name and contract.
- Keep commands and queries conceptually separate: a query returns information without hidden mutation; a command performs a state change and reports failure explicitly.
- Extract a function when the extracted name explains a meaningful concept, not merely to satisfy a line-count target.
- Do not use an arbitrary maximum such as 20 lines as a hard rule. Split when mixed responsibilities, nested branches, repeated detail, or changing abstraction levels make the behavior difficult to verify. ^[inferred]

## Cohesive classes and modules

- A class should have one coherent reason to change.
- Keep state with the rules that protect it; avoid anemic objects when invariants belong to the object.
- Keep application orchestration, domain rules, persistence, configuration, and presentation projections in their proper boundaries.
- Prefer several capability-focused interfaces over one broad facade that accumulates unrelated operations.
- Organize folders and namespaces by stable responsibility and module ownership, not by an arbitrary universal template.
- Avoid "god objects," generic `Manager` classes, and utility collections that become dumping grounds.
- A module should expose the smallest public surface required by its callers.
- Follow [[architecture/clean-architecture]] for dependency direction and boundary ownership.

## Dependencies and change boundaries

- Make dependencies explicit through constructors or operation parameters.
- Depend on the smallest stable abstraction needed by the current layer.
- Wrap clocks, filesystems, environment values, external services, and other uncontrollable dependencies behind replaceable boundaries.
- Use dependency injection to compose collaborators; do not hide service location inside business logic.
- Apply the open/closed principle at confirmed variation points: add a new implementation behind a stable abstraction when new variants are expected.
- Do not introduce inheritance, interfaces, factories, or plugin mechanisms for hypothetical future changes. YAGNI takes priority over speculative extensibility. ^[inferred]
- Prefer composition over inheritance unless an actual substitutable "is-a" relationship and stable base contract exist.

## Data and object boundaries

- Objects encapsulate behavior and protect invariants; data-transfer objects carry data without pretending to own domain behavior.
- Avoid hybrids that expose mutable internal data while also claiming to enforce rules.
- Do not expose collections that callers can mutate behind the owner's back.
- Keep navigation across object graphs shallow. Ask the responsible object for a result instead of retrieving internals and making decisions elsewhere.
- Represent meaningful identifiers, units, states, and validated groups with domain types when primitive values would permit invalid combinations.
- Use nullable reference annotations and explicit absence models; do not rely on undocumented `null` behavior.

## Error handling

- Make the normal flow readable and keep failure behavior explicit.
- Throw or return errors that identify the failed operation and stable reason.
- Catch only errors the current boundary can recover from, translate, compensate for, or enrich meaningfully.
- Preserve the original cause and stack when translating exceptions.
- Do not return `null`, `false`, empty collections, or default objects to disguise failures.
- Validate inputs at the boundary where their meaning is known.
- Keep cleanup and compensation deterministic; report compensation failures rather than hiding partial state.
- Error messages should provide actionable context without leaking secrets.

## Comments and documentation

- Prefer code that explains what it does through names and structure.
- Comments explain why a non-obvious constraint, trade-off, compatibility rule, or workaround exists.
- Do not restate the statement immediately below the comment.
- Delete obsolete comments and commented-out code.
- Use XML documentation for public contracts when callers need ownership, lifecycle, error, threading, cancellation, or compatibility information.
- A long comment that explains tangled behavior is a prompt to simplify the code first.

## Duplication and constants

- Remove duplicated knowledge, not merely similar-looking syntax.
- Extract shared logic only when the copies represent the same rule and should change together.
- Keep independent concepts separate even if their current implementations happen to look alike.
- Name business limits, protocol values, units, and configuration keys instead of scattering magic values.
- Do not replace every literal with a constant; values that are self-evident and local to one expression can remain literals. ^[inferred]
- Put deployable or environment-specific values in configuration rather than source.

## Tests enable clean changes

- Tests document behavior and make small refactoring safe.
- Difficult unit setup is a design signal: hidden dependencies, broad responsibilities, or tight coupling may need to be separated.
- Test public or internal behavioral contracts rather than private implementation details.
- Keep tests fast, isolated, deterministic, self-checking, and readable.
- Do not use coverage percentage as a substitute for meaningful behavior and error-path tests.

## Continuous cleanup

- Leave touched code slightly clearer than it was before, but keep cleanup relevant to the current change.
- Refactor in small, behavior-preserving steps with tests passing between steps.
- Remove dead code instead of commenting it out; version control retains history.
- Fix misleading names, obvious duplication, and local complexity when they are directly coupled to the current task.
- Do not expand a focused change into an unrelated repository-wide rewrite.
- Make technical debt explicit when safe cleanup exceeds the current scope.

## Common smells and responses

## Practical limits

Clean-code principles are decision aids, not mechanical scoring rules:

- Shorter is not automatically clearer.
- More classes and interfaces are not automatically cleaner.
- DRY does not justify coupling unrelated concepts.
- Open/closed does not require predicting every future extension.
- Abstraction is valuable when it names a stable concept or isolates a real source of change.
- Performance-sensitive code may need a less abstract shape, but the trade-off must be measured and documented.
- Repository consistency matters, but an existing pattern should not be copied when it violates correctness or a stable architecture boundary.

## Review checklist

- [ ] Names reveal domain meaning, ownership, state, and units.
- [ ] Each function and class has one cohesive responsibility and abstraction level.
- [ ] Public surfaces and parameter lists are no larger than required.
- [ ] Dependencies, side effects, mutation, and failure paths are explicit.
- [ ] Domain invariants remain with their owning object or module.
- [ ] Duplication represents shared knowledge before it is extracted.
- [ ] Comments explain rationale rather than restating code.
- [ ] No dead code, speculative abstraction, silent fallback, or broad catch was added.
- [ ] Tests protect observable behavior and permit safe refactoring.
- [ ] The touched area is at least as understandable as before the change.

## Related

- [[standards/csharp/csharp-coding-standard]]
- [[architecture/clean-architecture]]
