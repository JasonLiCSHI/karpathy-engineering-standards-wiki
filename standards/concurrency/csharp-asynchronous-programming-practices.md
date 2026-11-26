---
title: C# Asynchronous Programming Practices
category: skills
tags: [csharp, async, task, cancellation, visibility/public]
sources:
  - https://learn.microsoft.com/zh-cn/dotnet/csharp/asynchronous-programming/
  - https://learn.microsoft.com/zh-cn/dotnet/csharp/asynchronous-programming/task-asynchronous-programming-model
  - https://learn.microsoft.com/zh-cn/dotnet/csharp/asynchronous-programming/async-scenarios
  - https://learn.microsoft.com/zh-cn/dotnet/csharp/asynchronous-programming/async-return-types
  - https://learn.microsoft.com/zh-cn/dotnet/csharp/asynchronous-programming/cancel-async-tasks-after-a-period-of-time
  - https://learn.microsoft.com/zh-cn/dotnet/csharp/asynchronous-programming/generate-consume-asynchronous-stream
summary: Enforceable C# async rules for TAP APIs, workload selection, task composition, cancellation, exceptions, context, streams, resource lifetime, testing, and review.
provenance:
  extracted: 0.87
  inferred: 0.13
  ambiguous: 0.00
base_confidence: 0.95
lifecycle: verified
lifecycle_changed: 2026-07-31
tier: core
created: 2026-07-31T10:01:52+08:00
updated: 2026-08-05T15:18:26+08:00
relationships:
  - target: "[[standards/concurrency/async-io-and-cancellation]]"
    type: extends
  - target: "[[standards/concurrency/concurrent-programming-practices]]"
    type: related_to
---

# C# Asynchronous Programming Practices

> Primary source: [Asynchronous programming with C#](https://learn.microsoft.com/zh-cn/dotnet/csharp/asynchronous-programming/) and its Microsoft Learn guidance for TAP, scenarios, return types, cancellation, and asynchronous streams.

Asynchronous programming prevents a thread from being occupied while an operation is waiting. It primarily improves responsiveness and scalability; it does not automatically make work parallel or faster.

## Mental model

- `Task` represents an operation that may still be running and eventually completes successfully, faults, or is canceled.
- `Task<T>` additionally carries a result of type `T`.
- Calling an async method starts executing it synchronously on the current context until it reaches an incomplete `await`.
- At an incomplete `await`, the method records its continuation, returns control to its caller, and resumes after the awaited operation completes.
- `await` does not block the current thread.
- `async` and `await` do not create a thread. They coordinate continuations around awaitable operations.
- Code after `await` is still part of the same logical operation, including `try`, `catch`, `finally`, locals, and control flow.
- An async method with no effective `await` executes synchronously and adds unnecessary state-machine overhead.

Do not reason about async code as "a method running on its own thread." Reason about a logical operation that may suspend and resume.

## Choose the right execution model

### I/O-bound work

For network, database, file, timer, device, or other wait-dominated operations:

- call the true asynchronous API;
- `await` the returned task;
- do not wrap the operation in `Task.Run`;
- propagate cancellation and deadlines to the underlying API.

A thread-pool thread provides no benefit while the operation is only waiting for external completion.

### CPU-bound work

For expensive computation:

- keep the computation synchronous if the caller can execute it directly;
- use `Task.Run` at an application boundary when offloading is required to keep a UI or other affinity thread responsive;
- measure whether the work is large enough to justify scheduling and context-switch overhead;
- use explicit parallel algorithms only when the work is safely partitionable and measured to benefit.

Reusable libraries should not hide CPU offloading inside an API merely to add an `Async` suffix. Let the application choose the execution policy unless the library contract explicitly owns it. ^[inferred]

## Async all the way

- Once a call path becomes asynchronous, propagate `Task` and `await` to its natural entry point.
- Do not use `.Result`, `.Wait()`, `Task.WaitAll`, `Task.WaitAny`, or `Thread.Sleep` in an async path.
- Prefer:
  - `await task`;
  - `await Task.WhenAll(...)`;
  - `await Task.WhenAny(...)`;
  - `await Task.Delay(...)`.
- Sync-over-async can deadlock a captured context, consume threads while doing no work, complicate exception handling, and reduce service scalability.
- If an immutable legacy synchronous boundary cannot be changed, isolate and review the bridge explicitly. Do not let it spread through application code.
- Never expose both sync and async APIs by implementing one as a blocking wrapper over the other without proving the execution and context behavior.

## TAP API design

### Return types

- Return `Task` for an operation with no result.
- Return `Task<T>` for an operation that produces one result.
- Return `IAsyncEnumerable<T>` when results arrive incrementally through repeated asynchronous work.
- Use `async void` only for event handlers required by a `void` delegate contract.
- Avoid custom awaitable return types unless implementing infrastructure with a demonstrated need.

### Naming and signatures

- Add the `Async` suffix to asynchronous methods unless an established interface or platform contract requires another name.
- Place `CancellationToken` last, normally with a default value only when cancellation is genuinely optional.
- Keep synchronous argument validation and asynchronous execution contracts predictable.
- If invalid arguments must throw before a task is returned, use a non-async validating wrapper that invokes an inner async implementation after validation.
- Document whether an operation may complete synchronously, captures affinity, supports cancellation, reports progress, streams results, or owns retries.

### `ValueTask` restraint

- Default to `Task` and `Task<T>`.
- Consider `ValueTask<T>` only on a measured, allocation-sensitive path where synchronous completion is common.
- Treat it as a specialized contract, not a cheaper spelling of `Task<T>`.
- Do not make callers depend on repeated awaiting, broad composition, or storage semantics that the chosen `ValueTask` form does not promise.
- Prefer converting to `Task` at a boundary when normal task composition is needed.

## Compose operations by dependency

### Sequential dependency

Await immediately when the next step requires the result:

```csharp
var metadata = await LoadMetadataAsync(cancellationToken);
return await BuildDirectoryAsync(metadata, cancellationToken);
```

### Independent operations

Start independent work before awaiting, then compose it:

```csharp
var metadataTask = LoadMetadataAsync(cancellationToken);
var foldersTask = LoadFoldersAsync(cancellationToken);

await Task.WhenAll(metadataTask, foldersTask);

return BuildDirectory(
    await metadataTask,
    await foldersTask);
```

- Use `Task.WhenAll` when all results are required.
- Use `Task.WhenAny` when results should be handled in completion order.
- After `await Task.WhenAny(...)`, await the completed task itself to retrieve its result and observe its failure.
- Materialize LINQ-created task sequences with `ToArray()` or `ToList()` before composition so task creation is deliberate and occurs once.
- Do not start duplicate operations through repeated deferred enumeration.
- Bound fan-out. `WhenAll` coordinates tasks but does not limit concurrency or protect an external dependency.
- Preserve required ordering; concurrent start is correct only for independent operations.

## Exceptions and task observation

- Exceptions raised by asynchronous execution are stored in the returned task.
- Awaiting a faulted task rethrows its exception at the await point, so use ordinary `try`/`catch` around awaited operations.
- Catch only exceptions the current layer can handle, translate, compensate, or enrich.
- Observe every started task. An unawaited task can lose failure, cancellation, completion, and lifetime control.
- Do not discard a task with `_ =` unless ownership is deliberately transferred to a component that observes failures and controls lifetime.
- A task returned from `WhenAll` can represent multiple failures. If all failures matter, inspect the completed aggregate task rather than assuming the first rethrown exception is the whole failure set.
- Do not convert exceptions or cancellation into success-shaped results.
- Keep cleanup in `finally`, `using`, or `await using` so suspension does not bypass it.

## Cancellation and timeouts

Cancellation is a cooperative protocol, not forced termination:

- accept a `CancellationToken` when useful work can be abandoned;
- pass the same token through every cancellable child operation;
- check the token at meaningful boundaries in CPU loops or between non-cancellable steps;
- let `OperationCanceledException` propagate unless the current boundary owns the cancellation result;
- distinguish caller cancellation, timeout, and operation failure when the public contract requires different behavior;
- dispose owned `CancellationTokenSource` instances;
- use linked token sources when operation lifetime depends on several cancellation owners;
- do not reuse a token source across unrelated operation lifetimes.

For timeout policy:

- prefer an API's native timeout/deadline support when it has the required semantics;
- otherwise schedule cancellation with `CancelAfter` or an equivalent scoped token source;
- define what happens to underlying work after the caller stops waiting;
- do not treat a timeout race as proof that the underlying operation has stopped.

Cancellation must preserve invariants. Decide whether partial results are discarded, returned explicitly, or committed atomically.

## Context, affinity, and continuations

- UI and other affinity-based code normally resumes on its captured context so it can safely update owned state.
- Library code should avoid depending on a caller's synchronization context.
- Use `ConfigureAwait(false)` deliberately in library or infrastructure code that does not need the captured context.
- Do not apply `ConfigureAwait(false)` mechanically where later code requires UI, request, or other affinity state.
- In environments without a custom synchronization context, context capture may not change thread affinity; do not build correctness on incidental thread identity.
- Keep synchronous work before the first `await` and between awaits short on a UI or request-sensitive thread.
- If a continuation mutates shared state, follow [[standards/concurrency/concurrent-programming-practices]]; `await` is not a synchronization mechanism for arbitrary shared data.

## Asynchronous streams

Use `IAsyncEnumerable<T>` when:

- elements arrive over time;
- the consumer can process early elements before the producer finishes;
- buffering the full result would increase memory or latency;
- cancellation should stop further production.

Producer rules:

- yield each element when it becomes valid;
- propagate cancellation into page reads or other underlying operations;
- release the asynchronous enumerator's resources deterministically;
- do not hide an unbounded producer behind the stream.

Consumer rules:

- use `await foreach`;
- process, persist, or forward each item without accumulating all items unless required;
- pass cancellation with the API's token parameter or `WithCancellation`;
- understand that enumerator disposal is asynchronous;
- decide whether a partially consumed stream is an acceptable result.

Use `Task<IReadOnlyList<T>>` when the caller needs one complete atomic result; use `IAsyncEnumerable<T>` when incremental delivery is the contract.

## Asynchronous resource lifetime

- Use `await using` for owned `IAsyncDisposable` resources.
- Await asynchronous flush, close, commit, and rollback operations when they are part of correctness.
- The creator of an owned asynchronous resource normally owns disposal.
- Do not return while an owned background operation can still access a disposed resource.
- Avoid holding locks, transactions, or scarce pooled resources across unrelated awaits.
- If state must remain protected across an asynchronous operation, redesign the state transition or use an async-compatible coordination primitive; do not use a thread-affine monitor across `await`.

## Progress and partial results

- Use progress reporting for status; use return values or streams for data.
- Keep progress values immutable and small.
- Document the context on which progress callbacks run.
- Throttle or coalesce high-frequency progress so reporting does not become the bottleneck.
- Do not let progress callbacks own core control flow or failure handling.
- Prefer an asynchronous stream when each progress item is itself a meaningful consumable result.

## Testing async code

- NUnit `[Test]` and `[TestCase]` entry methods are synchronous `void`; they must not use `async` or return `Task`.
- Put asynchronous behavior and its assertions in a private `async Task` helper, then invoke it from the test entry with `.GetAwaiter().GetResult()`.
- Never use `async void`, `.Wait()`, or `.Result` in tests. The synchronous test-entry bridge is the only approved sync-over-async boundary.
- Await the system under test inside the helper so failures and completion are observed before the test entry returns.
- Test success, fault, cancellation, timeout, and cleanup paths.
- Verify tokens reach the dependency that performs the wait.
- Use controllable tasks, gates, or test doubles instead of `Thread.Sleep` and timing guesses.
- Test ordering only when ordering is part of the contract.
- For fan-out, test partial failure and prove that every started task is observed.
- For asynchronous streams, test empty, partial, complete, canceled, faulted, and early-disposal consumption.
- For affinity-sensitive code, test with the relevant host or synchronization context rather than assuming thread-pool behavior.
- Keep timeouts as test safety bounds, not as the mechanism that drives the scenario.

## Common async smells

## Review checklist

- [ ] The operation is correctly classified as I/O-bound, CPU-bound, or mixed.
- [ ] I/O uses true async APIs without unnecessary `Task.Run`.
- [ ] CPU offloading is owned by the correct application boundary and justified.
- [ ] The call path stays asynchronous without sync-over-async.
- [ ] Return type and `Async` naming communicate the contract.
- [ ] `async void` is limited to required event handlers.
- [ ] Independent work is composed deliberately and fan-out is bounded.
- [ ] Every started task has an observer and lifetime owner.
- [ ] Exceptions, multiple failures, cancellation, and timeout remain distinguishable.
- [ ] Cancellation is propagated to underlying work and preserves invariants.
- [ ] Context capture and affinity are intentional.
- [ ] Async streams and async disposal are used only when their lifecycle fits.
- [ ] Tests await behavior and cover failure, cancellation, timeout, and cleanup.

## Related

- [[standards/csharp/csharp-coding-standard]]
- [[standards/concurrency/concurrent-programming-practices]]
- [[standards/conventions/clean-code-practices]]
- [[architecture/clean-architecture]]
- [[architecture/pre-development-design-document-standard]]
