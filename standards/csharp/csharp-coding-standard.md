---
title: C# Coding Standard
category: skills
tags: [csharp, clean-code, code-review, conventions, visibility/public]
sources:
  - https://learn.microsoft.com/zh-cn/dotnet/csharp/fundamentals/coding-style/coding-conventions
  - https://learn.microsoft.com/zh-cn/dotnet/csharp/asynchronous-programming/
created: 2026-07-30T16:49:54+08:00
updated: 2026-10-15T15:56:00+08:00
summary: Enforceable C# and clean-code rules covering language use, design clarity, naming, exceptions, async code, LINQ, ReSharper cleanupcode/inspectcode with shared .DotSettings, and reviews.
provenance:
  extracted: 0.82
  inferred: 0.18
  ambiguous: 0.00
base_confidence: 0.94
lifecycle: verified
lifecycle_changed: 2026-07-30
tier: core
relationships:
  - target: "[[standards/csharp/modern-csharp-standard]]"
    type: extends
  - target: "[[standards/conventions/clean-code-practices]]"
    type: related_to
---

# C# Coding Standard

> Primary source: [.NET coding conventions - C#](https://learn.microsoft.com/zh-cn/dotnet/csharp/fundamentals/coding-style/coding-conventions)

## Purpose and priority

This standard turns the Microsoft conventions into rules that can be used during implementation, review, and CI. Its priorities, in order, are:

1. correctness and explicit failure behavior;
2. readability and accurate domain language;
3. consistency within the repository;
4. maintainability and safe evolution;
5. concise syntax when it does not weaken the first four goals.

The Microsoft page describes conventions used by documentation and sample repositories. It is a strong baseline, not an exhaustive or universally authoritative list. Project architecture, compatibility requirements, and a repository's checked-in `.editorconfig` take precedence when they intentionally differ.

Language-agnostic naming, function, class, dependency, refactoring, and duplication rules are defined in [[standards/conventions/clean-code-practices]] and are part of this standard.

## Enforcement

- Store formatting and style rules in a repository-level `.editorconfig` and a shared ReSharper `.DotSettings` layer (when used).
- Enable the .NET analyzers appropriate to the target framework and project type.
- **ReSharper `cleanupcode` & `inspectcode` workflow (mounting shared `.DotSettings`)**:
  - Whenever an agent or developer adds or modifies any `.cs` file, they **must** run both ReSharper `cleanupcode` and `inspectcode` with the repository's shared `.DotSettings` file explicitly mounted before considering the task complete:
    1. **Format with `cleanupcode`**: Run `jb cleanupcode <ProjectOrSolution> --settings="<shared>.DotSettings" --profile="Safe Cleanup" --include="<modified-cs-files>"` scoped strictly to the added or modified `.cs` files. Never format the entire solution or overwrite unrelated user changes, and never substitute a generic formatter that ignores `.DotSettings`.
    2. **Verify with `inspectcode`**: Run `jb inspectcode <ProjectOrSolution> --settings="<shared>.DotSettings" --include="<modified-cs-files>" --output="<report-path>"` and resolve all reported `WARNING` / `ERROR` inspections on the touched files.
    3. **Re-validate build and tests**: Always re-run the affected build and unit tests after `cleanupcode`, because moving `using` directives or shortening references can change symbol resolution.
    4. **Report blockers explicitly**: If `cleanupcode` or `inspectcode` is unavailable in the environment, report the blocker explicitly rather than claiming `.DotSettings` cleanup/inspection was performed.
- Run analyzers in CI so a rule is enforced consistently rather than depending on individual IDE settings.
- Assign diagnostics deliberate severities: correctness and safety rules should fail the build; subjective style rules can begin as suggestions or warnings.
- Do not add suppressions merely to make CI green. A suppression must be narrow and explain why the rule is incorrect for that location.
- Determine the effective C# language version before requiring modern syntax. Follow [[standards/csharp/csharp-language-version-aware-coding]].

Suggested baseline:

```editorconfig
[*.cs]
indent_style = space
indent_size = 4
tab_width = 4

csharp_new_line_before_open_brace = all
csharp_style_namespace_declarations = file_scoped:suggestion
csharp_using_directive_placement = outside_namespace:suggestion

dotnet_style_predefined_type_for_locals_parameters_members = true:suggestion
csharp_style_var_when_type_is_apparent = true:suggestion
csharp_style_var_elsewhere = false:suggestion
```

The repository remains the source of truth for exact rule values and severities.

## Language and type usage

### Modern language features

- Prefer current, non-obsolete language features when the effective language version supports them and they improve safety or clarity.
- Follow [[standards/csharp/csharp-language-version-aware-coding]] and [[standards/csharp/csharp-modern-language-practices]] before adopting newer syntax, especially for .NET 10 / C# 14 projects.
- Do not modernize unrelated code mechanically.
- Use C# keywords for built-in types: `string`, `int`, `bool`, and `object`, not `System.String`, `System.Int32`, `System.Boolean`, or `System.Object`.
- Prefer `int` for ordinary counts, indexes, and arithmetic. Use unsigned types only when the domain or external binary contract requires them.
- Use short-circuiting `&&` and `||` for conditional logic. Use `&` and `|` only when eager evaluation or bitwise behavior is intentional.
- Prefer records, init-only state, `required`, pattern matching, raw strings, and collection expressions when they encode the domain more clearly than older constructs.

### `var`

Use `var` when the type is evident from the right-hand side:

```csharp
var order = new Order();
var count = 10;
var names = orders.Select(order => order.Name);
```

Use an explicit type when the assignment hides the type:

```csharp
OrderSummary summary = repository.FindById(orderId);
int retryCount = settings.GetRetryCount();
```

Additional rules:

- Do not encode the type in the variable name, such as `inputInt` or `orderList`.
- Use names that express meaning and role, such as `retryCount`, `selectedOrders`, or `ownerFolder`.
- `var` is not a substitute for `dynamic`; use `dynamic` only for a deliberate runtime-binding boundary.
- LINQ projections and anonymous types normally use `var`.
- In `foreach`, prefer an explicit element type when it makes the collection element type clearer.

## Naming

- Names must express domain meaning rather than implementation mechanics.
- Use PascalCase for record primary-constructor parameters because they become public properties.
- Use camelCase for class and struct primary-constructor parameters.
- Private instance fields must start with `my`, followed by a PascalCase role name, such as `myRepository`, `myStore`, or `myDisposed`.
- Constants (`const`) and `static` fields (including `private static` and `private static readonly`) do not need to start with `my` and must start with an uppercase letter (PascalCase), such as `SyncLock`, `GlobalRegistry`, or `DefaultWorkerThreadName`.
- Non-private instance member variables (`public`, `internal`, `protected`, `protected internal`, `private protected`) must be exposed as properties rather than fields.
- Use PascalCase aliases for anonymous-type properties.
- Rename ambiguous projected members. Prefer `CustomerName` and `DistributorName` over two unrelated properties both called `Name`.
- Do not introduce a second meaning for an established domain term. Architectural naming follows [[architecture/clean-architecture]] and the owning module's ubiquitous language.

The source page does not define a complete public API naming system. Existing repository conventions remain authoritative for type, member, interface, field, and test names.

## Construction and initialization

- Use target-typed `new()` or `var value = new Type()` when the constructed type remains obvious.
- Use object initializers when they make one-time initialization easier to read.
- Use `required` properties for object-initializer models whose members must be supplied.
- Do not replace constructors with `required` properties when a constructor must validate cross-field invariants or prevent partially valid objects. ^[inferred]
- Use collection expressions when the language version supports them and the target type is clear:

```csharp
string[] categories = ["Cardiac", "Neuro", "Trauma"];
```

## Strings

- Use string interpolation for short, expression-based composition.
- Prefer expression names over positional formatting placeholders.
- Use raw string literals for multiline text, JSON, XML, regular expressions, or content that would otherwise require heavy escaping.
- Use `StringBuilder` for repeated or large string accumulation, especially inside loops.
- Do not introduce `StringBuilder` for a small, fixed number of concatenations when interpolation is clearer.

## Delegates, events, and static members

- Prefer `Func<>` and `Action<>` over declaring a custom delegate when no domain-specific delegate type is required.
- Use method-group conversion instead of explicitly constructing a delegate.
- A lambda event handler is acceptable only when it never needs to be unsubscribed.
- Use a named handler, or retain the delegate instance, when lifecycle cleanup requires unsubscription.
- Access static members through the type that declares them.
- Do not qualify an inherited static member through a derived type; it hides ownership and can become ambiguous.

## Exceptions and resource management

- Catch only exceptions that the current layer can handle, translate, enrich, or compensate for correctly.
- Use specific exception types and meaningful messages.
- Do not catch `Exception` without a narrow, justified boundary and suitable filtering.
- Rethrow with `throw;` when preserving the original stack trace.
- Do not swallow failures or return success-shaped fallback values.
- Use `using` declarations or statements for `IDisposable` resources when disposal is the only purpose of `finally`.
- Use `await using` for `IAsyncDisposable`.
- Keep resource ownership explicit: the code that creates an owned resource is normally responsible for disposing it.

## Async and concurrency

[[standards/concurrency/csharp-asynchronous-programming-practices]] is the authoritative async implementation and review standard. At minimum:

- Use true asynchronous APIs for I/O-bound work; do not wrap I/O in `Task.Run`.
- Use `Task.Run` for CPU-bound work only when the application boundary deliberately owns offloading.
- Propagate `await`, cancellation, exceptions, and lifetime to the natural entry point.
- Do not block asynchronous work with `.Result`, `.Wait()`, `WaitAll`, `WaitAny`, or `Thread.Sleep`.
- Name asynchronous methods with the `Async` suffix unless an established contract dictates another name.
- Return `Task` or `Task<T>`; reserve `async void` for event handlers and `ValueTask<T>` for measured specialized paths.
- Bound concurrent fan-out and observe every started task.
- Choose synchronization-context capture and `ConfigureAwait(false)` deliberately.
- Use `IAsyncEnumerable<T>` only when incremental asynchronous delivery is part of the contract.

## LINQ and collection operations

- Use LINQ when it makes filtering, projection, grouping, or aggregation easier to understand.
- Use meaningful query and range-variable names.
- Filter early so later clauses operate on the smallest relevant set.
- Give anonymous projected properties explicit PascalCase aliases when their meaning would otherwise be ambiguous.
- Prefer a straightforward loop when LINQ would hide mutation, repeated enumeration, exception behavior, or important performance characteristics. ^[inferred]
- Be aware of deferred execution and of changes between `IEnumerable<T>` and `IQueryable<T>`; they can change where and how a query executes.
- Do not enumerate the same expensive query repeatedly without a deliberate reason.
- Avoid unnecessary allocation-heavy LINQ in hot paths; when the code is a measurable hotspot, use loops and span-oriented patterns instead and see [[standards/performance/csharp-performance-practices]].

## Namespaces and imports

- Prefer file-scoped namespaces when a file contains one namespace and the effective language version supports them.
- Place `using` directives outside the namespace declaration.
- This avoids context-dependent name resolution when similarly named nested namespaces are introduced later.
- Use `global::` only to resolve a real ambiguity, not as routine noise.

## Formatting and layout

- Use four spaces for indentation; do not use tab characters.
- Use Allman braces: opening and closing braces appear on their own lines and align with the containing declaration or statement.
- Write one statement per line.
- Write one declaration per line.
- Add blank lines between logically separate members and blocks.
- Break long expressions across lines where the break exposes structure; keep continuation indentation consistent.
- Prefer parentheses when they make compound conditions easier to verify.
- Let the formatter own mechanical layout. Do not hand-align code in a way that creates noisy diffs after neighboring names change.
- Always execute the repository's configured formatter and static analyzer (such as `dotnet format` or JetBrains `cleanupcode` / `inspectcode` with the shared `.DotSettings` / `.editorconfig`) on all added or modified `.cs` files.

Microsoft's 65-character line limit is specific to documentation examples displayed on mobile screens. It is not imposed on production code by this standard. A production repository may set its own line-length guidance; clarity is the deciding rule. ^[inferred]

## Comments and XML documentation

- Prefer self-explanatory names and structure over comments that restate the code.
- Use `//` for short explanations of intent, constraints, or non-obvious behavior.
- Put comments on their own line above the code they explain.
- Insert one space after `//`; write complete, punctuated sentences.
- Avoid long `/* ... */` explanations in implementation code.
- Document public APIs with XML documentation when callers need contract, parameter, return-value, exception, ownership, threading, or lifecycle information.
- Keep documentation synchronized with behavior. Stale comments are defects.
- Do not use comments to excuse unclear code that can be simplified.

## Security

- Apply the .NET secure-coding guidance in addition to style rules.
- Validate data at trust boundaries.
- Do not expose secrets in source, logs, exceptions, test data, or configuration committed to the repository.
- Avoid constructing commands, paths, queries, or markup through unchecked string concatenation.
- Treat security analyzer diagnostics as correctness findings, not cosmetic style warnings.

## Review checklist

Before approving C# code, verify:

- [ ] The effective language and target-framework versions were considered.
- [ ] The change follows [[standards/conventions/clean-code-practices]] without applying its heuristics mechanically.
- [ ] Names use domain language and do not redefine established concepts.
- [ ] `var`, modern syntax, LINQ, and lambdas improve rather than obscure readability.
- [ ] Exceptions are specific, handled at the correct boundary, and never silently swallowed.
- [ ] Disposable and asynchronous lifecycles are explicit.
- [ ] Async code does not block or introduce avoidable context/deadlock risks.
- [ ] Namespace, formatting, and comment rules match `.editorconfig` and the shared `.DotSettings`.
- [ ] ReSharper `cleanupcode` (`--profile="Safe Cleanup"`) and `inspectcode` were run with the shared `.DotSettings` mounted on all added or modified `.cs` files, and the build and unit tests were re-verified afterward.
- [ ] Public contracts document important errors, ownership, threading, and lifecycle behavior.
- [ ] Analyzer output is clean or narrowly justified.

## Related

- [[standards/conventions/clean-code-practices]]
- [[standards/concurrency/csharp-asynchronous-programming-practices]]
- [[standards/concurrency/concurrent-programming-practices]]
- [[standards/csharp/csharp-language-version-aware-coding]]
- [[architecture/clean-architecture]]
