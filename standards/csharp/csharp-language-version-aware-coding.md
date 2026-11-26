---
title: C# Language-Version-Aware Coding
category: skills
tags: [csharp, clean-code, language-version, code-review, testing, visibility/public]
sources: [distilled-engineering-standards]
summary: C# projects should actively evaluate useful features available in their effective language version, while adopting them only when they improve semantics, safety, or clarity.
provenance:
  extracted: 0.76
  inferred: 0.22
  ambiguous: 0.02
base_confidence: 0.9
lifecycle: verified
lifecycle_changed: 2026-07-22
tier: core
created: 2026-07-22T13:31:45+08:00
updated: 2026-07-22T13:34:26+08:00
relationships:
  - target: "[[standards/csharp/modern-csharp-standard]]"
    type: extends
  - target: "[[standards/csharp/csharp-modern-language-practices]]"
    type: related_to
---

# C# Language-Version-Aware Coding

## Required rule

For every C# project, implementation and code review must determine the effective language version and explicitly consider whether features available up to that version can improve the code. This is an evaluation requirement, not a requirement to replace working syntax merely because a newer form exists.

## Determine the effective language version first

Before recommending a feature:

1. Check the project-level `LangVersion`.
2. Check imported `Directory.Build.props` and other shared build files.
3. If `LangVersion` is not explicit, determine the compiler default from the target framework and SDK.
4. Distinguish compiler support from target-framework/BCL support; a language feature may compile while its generated/runtime support still requires compatibility work.
5. Apply the conventions of the narrowest project being changed rather than assuming every project in a solution uses the same version.

## Language-feature checklist

The following examples are not exhaustive. Review the stable features introduced between the project's existing coding baseline and its effective language version.

| Feature | Prefer when | Avoid when |
|---|---|---|
| Records | The type is an immutable value, message, snapshot, or DTO and value equality is intentional. | The type is a mutable entity or aggregate, or changing equality semantics would be surprising. |
| Init-only properties | State should be assigned during construction/object initialization and remain stable afterward. | The object has a legitimate mutation lifecycle. |
| Target-typed `new` | The target type is obvious from the declaration, parameter, property, or return context. | Removing the type makes domain-heavy construction harder to read. |
| Enhanced pattern matching | `and`, `or`, `not`, relational, or property patterns express a rule more directly than repeated boolean branches. | The pattern is denser or less familiar than the original condition. |
| Static lambdas/local functions | Capture is unnecessary and preventing accidental closure allocation or hidden dependencies is useful. | Adding `static` creates noise without clarifying capture behavior. |
| Covariant returns | A derived API can return a more specific type without adapter casts. | The hierarchy does not benefit or the public contract becomes less obvious. |
| Top-level statements | A small executable, sample, migration tool, or focused harness benefits from minimal ceremony. | Production libraries and substantial applications need an explicit entry-point structure. |

## Review standard

- Prefer features that encode domain semantics, prevent invalid mutation, remove accidental captures, or reduce branching.
- Do not perform broad syntax-only modernization in unrelated code.
- Preserve repository conventions inside one cohesive change set.
- Treat conversion from class to record as a behavioral change because equality, printing, and copy semantics change.
- Build and test after language-feature adoption, including contract projects that target older frameworks.
- For the current repository baseline, treat .NET 10 / C# 14 as the modern target, while still respecting the effective per-project `LangVersion`.
- If an available language feature was considered but rejected, no comment is normally required; document the reason only when the choice is non-obvious or likely to be revisited.

## Test code

The same rule applies to tests, with readability as the strongest constraint:

- Use records or init-only test data only when they make fixture intent clearer.
- Use target-typed construction when the expected type remains obvious.
- Use static lambdas when capture prevention matters, not on every assertion projection.
- Do not let syntax modernization weaken three-part names, Given-When-Then flow, explicit expected values, or meaningful assertions.

## Related

- [[standards/csharp/csharp-modern-language-practices]]
- [[standards/performance/csharp-performance-practices]]
- [[architecture/clean-architecture]]
