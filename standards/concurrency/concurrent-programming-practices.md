---
title: Concurrent Programming Practices
category: skills
tags: [concurrency, thread-safety, synchronization, performance, testing, visibility/public]
sources:
  - https://itmyhome.com/java-concurrent-programming-combat/java-concurrent-programming-combat.pdf
summary: Language-agnostic rules for shared-state safety, task execution, cancellation, liveness, scalability, memory visibility, synchronization, and concurrent testing.
provenance:
  extracted: 0.84
  inferred: 0.16
  ambiguous: 0.00
base_confidence: 0.93
lifecycle: verified
lifecycle_changed: 2026-07-31
tier: core
created: 2026-07-31T09:58:28+08:00
updated: 2026-07-31T10:01:52+08:00
relationships:
  - target: "[[standards/concurrency/thread-safety-and-parallelism]]"
    type: extends
  - target: "[[standards/concurrency/csharp-asynchronous-programming-practices]]"
    type: related_to
---

# Concurrent Programming Practices

> Source: Brian Goetz et al., *Java Concurrency in Practice*, using the supplied [Chinese PDF](https://itmyhome.com/java-concurrent-programming-combat/java-concurrent-programming-combat.pdf). The practices below generalize the book's design principles beyond Java and do not reproduce its text or API-specific examples.

Concurrent programming coordinates independent activities that overlap in time. Correctness requires more than preventing simultaneous writes: shared state must preserve invariants, updates must become visible in a defined order, work must eventually progress, and lifecycle transitions must remain controllable.

## Start with the reason for concurrency

- Introduce concurrency only for a concrete goal: responsiveness, throughput, latency hiding, isolation of blocking work, or parallel use of processors.
- Distinguish concurrency from parallelism. A system may coordinate concurrent work on one processor or execute work in parallel on many processors.
- Prefer a serial design when it meets requirements. Concurrency adds state-space, scheduling, shutdown, failure, and testing complexity.
- Decompose the problem into independent tasks before choosing threads, executors, event loops, actors, or async runtimes.
- State which work may overlap, which work must remain ordered, and which result wins when operations race.
- Treat scheduling as nondeterministic unless a synchronization contract establishes ordering.

## Correctness dimensions

A concurrent design must address all three:

- **Safety:** nothing invalid happens; invariants, atomicity, ownership, and visibility remain correct.
- **Liveness:** useful work eventually progresses; the design avoids deadlock, starvation, livelock, and unbounded waiting.
- **Performance:** concurrency improves the required metric without unacceptable contention, queueing, context switching, or resource use.

Performance never compensates for a safety or liveness defect.

## Classify every piece of state

Choose one ownership model deliberately:

1. **Immutable:** construct completely, publish safely, and never mutate.
2. **Thread/task confined:** one execution context owns the state; other contexts communicate by messages or submitted tasks.
3. **Shared and synchronized:** all access follows one documented synchronization protocol.
4. **Transferred:** ownership moves through a queue, channel, future, or other handoff and the sender stops accessing it.

- Prefer immutable or confined state before shared mutable state.
- Minimize the amount of state that is shared and the duration for which it is shared.
- Do not expose mutable internals that let callers bypass the owner's protocol.
- Treat callbacks, iterators, views, and returned collections as possible state leaks.
- If ownership is unclear, the design is incomplete.

## Invariants, atomicity, and race conditions

- Identify invariants across fields and objects before selecting synchronization.
- Operations that read, decide, and update form one compound action and must be atomic relative to competing operations.
- Protect every variable participating in one invariant with the same synchronization policy.
- A collection of individually atomic variables does not make a multi-variable invariant atomic.
- Check-then-act, read-modify-write, lazy initialization, and "add if absent" are common race sites.
- Prefer a primitive that implements the complete atomic operation over composing separate thread-safe calls.
- Keep synchronization large enough to protect the invariant and small enough to exclude slow unrelated work.
- Never weaken correctness merely to reduce locking.

## Visibility, ordering, and safe publication

- Synchronization coordinates both mutual exclusion and visibility.
- Without an ordering edge, another execution context may observe stale state, partially initialized state, or operations in an unexpected order.
- Publish a mutable object only through a mechanism that guarantees its reference and initialized state become visible together.
- After publication, every later mutation still needs the object's synchronization protocol.
- Immutable or effectively immutable objects are easiest to share, but they must be fully constructed before becoming reachable.
- Do not let an object escape from its constructor through callbacks, registration, task startup, global storage, or overridable methods.
- Prefer standard channels, queues, futures, immutable messages, locks, or runtime-supported atomic publication over hand-written memory-ordering tricks.
- Document the language/runtime memory guarantees on which any lock-free or low-level design depends.

## Encapsulate synchronization

- A thread-safe component owns its mutable state and its synchronization policy.
- Document whether a public operation is thread-safe, atomic, blocking, ordered, reentrant, cancellable, or restricted to one execution context.
- Document which lock or mechanism protects each state group for maintainers.
- Do not require callers to guess or imitate a component's internal locking.
- Prefer composition and delegation when adding atomic behavior to an existing component; external locking against undocumented internals is fragile.
- Do not hold an internal lock while invoking unknown or overridable code.
- Return snapshots, immutable values, or controlled handles instead of live mutable state.
- A component not explicitly documented as thread-safe should be treated as not thread-safe.

## Prefer high-level concurrency building blocks

- Prefer tested queues, channels, futures/promises, semaphores, latches, barriers, task groups, concurrent collections, and cancellation tokens over raw wait/notify or ad hoc flags.
- Select a primitive whose semantics match the problem: capacity, one-time completion, phase coordination, permit control, result delivery, or producer-consumer handoff.
- Use bounded queues or channels when producers can outrun consumers.
- Define the saturation policy: block, reject, shed, coalesce, sample, persist, or degrade.
- Do not use sleep as a coordination mechanism.
- Avoid busy waiting unless the wait is proven extremely short and the runtime/hardware contract is explicit.
- Build a custom synchronizer only when existing primitives cannot express the required state transition.

## Separate tasks from execution policy

- A task describes work; an execution policy decides where, when, how many, and with what priority or resource budget tasks run.
- Keep task code independent of thread creation and pool management.
- Use managed, bounded execution rather than creating an unbounded execution context per request.
- Size concurrency from workload characteristics, resource limits, latency goals, and measured blocking—not from a universal formula.
- Account for dependencies between tasks. A bounded pool can deadlock if tasks wait for subtasks queued behind them.
- Long-running, blocking, CPU-bound, latency-sensitive, and affinity-bound tasks may need different execution policies.
- Make queue capacity, admission, rejection, deadlines, and observability part of the service contract.
- Preserve task failures; do not let background exceptions disappear.

## Structured lifecycle, cancellation, and shutdown

- Cancellation is cooperative. The requester signals intent; the task decides where it can stop while preserving invariants.
- Define cancellation checkpoints in loops, blocking calls, and multi-stage workflows.
- Propagate cancellation and deadlines to child work and external operations.
- Do not catch and discard cancellation/interruption signals. Either terminate, translate them consistently, or restore/propagate the signal.
- Make cleanup deterministic with scoped resource management or `finally`-equivalent mechanisms.
- Distinguish graceful shutdown from immediate shutdown:
  - graceful shutdown stops admission and completes accepted work;
  - immediate shutdown requests cancellation and reports work that did not complete.
- The component that creates an executor, scheduler, worker, or channel owns its shutdown.
- Define lifecycle states and legal transitions such as created, running, stopping, terminated, and failed.
- Coordinate dependent service shutdown in a known order; concurrent cleanup callbacks can race or deadlock.
- Never rely on finalization, process termination, or daemon/background status to release durable resources.

## Single-threaded subsystems and affinity

- Thread confinement is often simpler than locking for UI models, native libraries, event loops, protocol state machines, and non-thread-safe components.
- Route every access through the owning scheduler or execution context.
- Keep work on the affinity thread short; move long-running or blocking work out and marshal only results back.
- Clearly separate the thread-safe submission API from the confined implementation.
- Do not block an affinity thread waiting on work that requires the same thread to finish.

## Avoid liveness failures

### Deadlock

- Establish and document a global lock order when multiple locks may be acquired.
- Prefer open calls: update protected state, release the lock, then call external code.
- Avoid blocking I/O, long computation, callbacks, waits, and task submission while holding a lock.
- Include hidden locks and external resources—database connections, files, sockets, pools, and permits—in deadlock analysis.
- Use timed acquisition only when timeout has defined recovery semantics; it does not repair an incoherent locking design.

### Starvation and responsiveness

- Bound monopolization of locks, workers, queues, and affinity threads.
- Avoid depending on thread priority for correctness or portability.
- Separate workloads when one class of work can consume all capacity needed by another.
- Measure queue wait and lock wait, not only execution time.

### Livelock

- Distinguish retryable failures from permanent failures.
- Use bounded retries, exponential backoff, jitter, and poison-work handling.
- Ensure compensation or retry changes the conditions that caused failure.
- Surface repeated non-progress as an operational failure.

## Performance and scalability

- Define the target metric first: throughput, response time, tail latency, utilization, fairness, or capacity.
- Measure before optimizing and retain a serial baseline.
- The serial fraction limits maximum speedup; adding workers cannot overcome serialized policy, exclusive resources, or coordination.
- Reduce contention by:
  - shortening critical sections;
  - moving slow work outside locks;
  - reducing lock granularity when invariants permit;
  - partitioning independent state;
  - using read-sharing, immutability, confinement, or non-blocking structures where justified.
- More workers can reduce performance through scheduling, cache effects, context switching, memory pressure, and contention.
- Asynchronous handoff can improve caller latency but introduces queueing, backpressure, delivery guarantees, failure propagation, and shutdown obligations.
- Do not add object pools or custom lock-free structures without measured benefit.
- Optimize common paths without making rare failure and cancellation paths incorrect.

## Explicit locks and condition waiting

- Prefer the simplest synchronization form that satisfies the contract.
- Use advanced lock features only for a real need such as timed acquisition, cancellable acquisition, multiple wait sets, read/write coordination, or non-block-scoped locking.
- Release explicit locks on every path using deterministic cleanup.
- Associate each condition wait with:
  - one protected state predicate;
  - the lock that protects that predicate;
  - a loop that rechecks the predicate after every wake-up.
- Change protected state before signaling waiters.
- Assume wake-ups may be early, unrelated, duplicated, or delayed.
- Encapsulate condition queues; callers should not coordinate directly with an object's internal wait set.

## Atomic and non-blocking algorithms

- Atomic read-modify-write primitives can reduce contention for simple independent state.
- A single atomic variable is not enough for invariants spanning several values unless they are represented by one immutable state snapshot.
- Compare-and-swap loops must recompute from the latest observed state after failure.
- Non-blocking algorithms are difficult to design, review, and prove; prefer mature library implementations.
- Consider ABA-style changes when equality can return to an old value; versioning or stamped state may be required.
- Lock-free does not mean wait-free, fair, simpler, or always faster.
- Use low-level atomics only with explicit memory-order, progress, overflow, reclamation, and failure analysis.

## Testing concurrent software

Correctness tests should cover:

- invariants under many interleavings;
- simultaneous starts using barriers or coordinated gates;
- blocking and unblocking behavior;
- cancellation before start, during execution, and during blocking;
- graceful and immediate shutdown;
- saturation, rejection, timeout, and backpressure;
- resource release and worker termination;
- failures in tasks, callbacks, and cleanup;
- deadlock or non-progress bounded by a test timeout.

Test practice:

- Repeat stress tests under varied worker counts, data sizes, timing perturbations, and supported runtime configurations.
- Avoid tests that depend on exact sleeps or one expected schedule.
- Make failures self-checking and preserve enough state, task, queue, and timing information to diagnose them.
- Use code review, static analysis, runtime diagnostics, tracing, and profiling as complements to tests.
- Recognize that instrumentation can change timing and hide or expose races.
- For performance tests, control warm-up, runtime optimization, garbage collection, input realism, contention level, dead-code elimination, and environmental noise.
- Compare distributions and tail latency, not only one average.

## Observability and operations

- Expose active work, queue depth, saturation, rejections, cancellations, retries, timeouts, lock/permit wait, completion rate, and shutdown state.
- Correlate parent and child work across asynchronous boundaries.
- Record failures from background tasks with actionable context.
- Provide thread/task dumps or equivalent scheduler diagnostics for non-progress incidents.
- Alert on sustained queue growth, repeated retries, worker exhaustion, and shutdown timeouts.
- Keep diagnostics safe under failure; logging must not introduce recursive blocking or deadlock.

## Concurrent API contract

For every public concurrent component, document:

- state ownership and mutability;
- thread/task safety and affinity requirements;
- atomic operations and consistency guarantees;
- ordering and visibility guarantees;
- whether methods may block and for how long;
- cancellation, deadline, retry, and interruption behavior;
- queue capacity and saturation behavior;
- callback execution context and whether locks are held;
- lifecycle ownership and shutdown guarantees;
- failure propagation and observability.

## Common concurrency smells

## Design-document requirements

For any change that introduces concurrency, document:

- the concurrency objective and why a serial design is insufficient;
- task decomposition, execution policy, capacities, and resource budgets;
- ownership and mutability classification for every shared state group;
- invariants, atomic operations, visibility, ordering, and publication guarantees;
- lock/condition protocol or message-flow protocol;
- cancellation, deadlines, failure propagation, shutdown, and cleanup;
- deadlock, starvation, livelock, saturation, and overload analysis;
- performance hypothesis, serial fraction, target metric, and measurement plan;
- deterministic tests, stress tests, diagnostics, and operational signals.

See [[architecture/pre-development-design-document-standard]] for the full implementation gate.

## Review checklist

- [ ] Concurrency solves a stated responsiveness, throughput, or isolation requirement.
- [ ] Task boundaries and execution policy are separate.
- [ ] Every state item is immutable, confined, transferred, or protected.
- [ ] Invariants and compound actions have one complete atomicity policy.
- [ ] Publication, visibility, and ordering guarantees are explicit.
- [ ] No slow, blocking, or unknown code runs while holding an internal lock.
- [ ] Capacity and saturation behavior are bounded and observable.
- [ ] Cancellation and shutdown are cooperative, propagated, and tested.
- [ ] Lock ordering and liveness risks have been reviewed.
- [ ] Performance changes are measured against a baseline and target metric.
- [ ] Tests vary interleavings and cover failure, overload, cancellation, and cleanup.
- [ ] The public concurrency contract is documented.

## Related

- [[standards/concurrency/csharp-asynchronous-programming-practices]]
- [[architecture/clean-architecture]]
- [[standards/conventions/clean-code-practices]]
- [[architecture/pre-development-design-document-standard]]
