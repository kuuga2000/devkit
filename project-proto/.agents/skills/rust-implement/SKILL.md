---
name: rust-implement
description: Implement and fix Rust libraries, binaries, CLIs, services, embedded software, and other Rust projects. Use for code changes rather than review-only requests.
---

# Implement Rust

Read repository instructions, the relevant requirements, Cargo manifests, toolchain configuration, affected modules, and tests. Follow the project's edition, MSRV, target platforms, dependencies, architecture, and error conventions. Discover whether this is a library, CLI, service, embedded/no_std application, FFI component, or workspace before choosing tools. Do not impose a runtime, framework, allocator, or standard-library dependency.

## Establish behavior

Identify inputs, observable outputs, failure behavior, compatibility constraints, and acceptance criteria. Separate requirements from assumptions. Resolve ambiguity that changes domain meaning, safety, or public compatibility before implementing that part; continue independent work. Do not add an approval ceremony when implementation is already authorized.

For the affected code, determine ownership and resource lifetimes, invariants, allowed state transitions, and limits. For public interfaces consider Rust API compatibility, trait bounds, feature flags, wire/file formats, CLI exit codes, or FFI layout as applicable. Do not add unrelated infrastructure or abstractions.

## Implement

Prefer safe Rust, clear ownership, and the smallest complete change consistent with the project. Borrow when appropriate; clone when shared independent ownership is actually needed. Use traits, generics, dynamic dispatch, and shared state only where their tradeoffs serve the requirements. Avoid adding bounds or lifetimes that unnecessarily restrict public APIs.

Use Result/Option and typed errors when callers distinguish failure classes; preserve useful error context at application boundaries. Recoverable failures should not become panics. Document intentional panic contracts and safety invariants. For unsafe/FFI code, establish validity, alignment, aliasing, lifetime, ownership transfer, and unwinding obligations; a SAFETY comment alone is not proof.

Choose numeric representations from domain requirements. Check overflow, narrowing casts, units, and serialization. Exact financial values may require decimals or scaled integers; scientific calculations may require floating point and explicit tolerance/NaN behavior.

Apply relevant checks only:

- **Async/concurrency:** choose synchronization from access patterns; avoid blocking runtime workers or holding locks across unrelated I/O. Bound tasks/queues and external calls; handle deadlines, retries, cancellation, task errors, and shutdown. A timeout does not guarantee underlying work stopped.
- **Services/storage:** validate boundaries, preserve public error contracts, protect secrets, use appropriate transactions, and handle cache isolation, stale data, and update ordering where required.
- **CLIs/files:** preserve exit codes, stdout/stderr conventions, platform/path behavior, and data integrity during interrupted or failed writes.
- **Libraries:** preserve supported features, MSRV, target compatibility, documentation examples, and downstream usability.
- **Embedded/no_std/FFI:** respect allocation, stack, timing, interrupt, ABI, hardware, and target constraints. Do not assume host tests establish device correctness.

## Idiomatic Rust and API quality

Use the [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/checklist.html) as a design checklist, not a reason to add unnecessary APIs or dependencies. Follow [Cargo's compatibility guidance](https://doc.rust-lang.org/cargo/reference/semver.html) for changes to published interfaces.

- Keep the public surface small and implementation details private. Protect invariants through constructors and methods rather than exposing mutable fields. Use newtypes or enums when they prevent real mistakes; introduce typestate only when its additional complexity pays off.
- Prefer standard conversion and collection conventions: From for infallible conversion, TryFrom for fallible conversion, AsRef for appropriate borrowed views, and iter/iter_mut/into_iter for iteration. Use slices and str for borrowed views when ownership is unnecessary; accept owned values when storing or consuming them avoids a copy. Do not force one parameter style onto every API.
- Implement common traits only when their semantics are correct. Keep Eq/Hash and ordering consistent; derive Copy only when implicit duplication is appropriate. Avoid exposing secrets through derived Debug. Consider must_use for values whose accidental discard is likely a bug, and non_exhaustive for extensible public types when compatible with the intended construction and matching API.
- Document changed public behavior with rustdoc and useful examples. Describe errors, intentional panics, and safety preconditions where applicable; run relevant doctests. Document blocking behavior, cancellation effects, and complexity when callers need that information.
- Use RAII for ordinary resource cleanup, while making fallible finalization explicit when callers must observe errors; Drop cannot return a Result or perform awaited cleanup. A safe public API must uphold its invariants for every input without relying on callers to obey undocumented unsafe preconditions.
- Keep Cargo features additive where practical, and check feature unification and default features when preserving no_std or minimal builds. Avoid enabling broad dependency features without need. Adding optional dependencies, changing trait bounds, or losing Send/Sync can affect downstream compatibility even when ordinary local tests pass. See [Cargo features](https://doc.rust-lang.org/cargo/reference/features.html).
- Prefer a clear algorithm and appropriate data structures before allocation tricks, inline annotations, custom atomics, or unsafe optimization. Measure representative release workloads before claiming a performance improvement; choose readable loops or iterators based on the task. Do not treat cloning, dynamic dispatch, or a synchronous mutex as inherently wrong without a concrete cost or correctness issue.
- Let rustfmt and configured Clippy lints enforce style. Investigate warnings and narrowly justify unavoidable exceptions; do not turn every pedantic lint into a universal rule. Respect the project's stable toolchain and MSRV instead of upgrading the edition or adopting nightly features merely to follow a newer example.

## Crate-specific guidance

Apply the sections matching dependencies already used or explicitly selected for the task. Inspect Cargo.toml, Cargo.lock, feature flags, and existing usage first. Follow APIs supported by the project's versions; consult version-matched official documentation when uncertain. Add dependencies or features only when the change needs them, and respect MSRV, target, TLS, and std/no_std constraints. These sections are conditional guidance, not a required stack.

### Tokio

- Reuse the application's runtime; avoid constructing or blocking on a nested runtime inside async execution. Enable the runtime, time, I/O, sync, and test features actually needed.
- Bound work before spawning or admitting it with permits, bounded channels, or a limited in-flight set. Collecting handles in a JoinSet alone does not bound concurrency. Choose queue-full behavior deliberately.
- Own task lifetimes with tracked handles or task sets. Handle both task join failures and inner operation errors. Dropping a JoinHandle detaches its task; connect worker cancellation to application shutdown.
- Check cancellation safety of operations used in select! or timeouts. Dropped futures may leave partial state; use a recoverable protocol for multi-step effects. Handle channel closure rather than polling a closed channel in a busy loop.
- Use spawn_blocking for suitable blocking work and bound CPU-heavy submissions, or use an appropriate CPU pool. Started blocking tasks cannot generally be aborted; plan cooperative termination for long-running work. An async mutex is needed only when its suspension semantics fit; a short synchronous critical section can use a synchronous mutex if it does not cross an await or cause unacceptable blocking.
- Shut down by stopping admission, notifying workers, and waiting within a defined budget. Test timing behavior with Tokio's controllable time when enabled instead of wall-clock sleeps.

### Axum and Tower

- Keep router construction testable independently of socket binding. Use typed extractors and explicit application state; place body-consuming extractors in the position required by the installed Axum version. Match route syntax to that version.
- Map domain failures to stable, sanitized IntoResponse results at the HTTP boundary. Account for extractor rejections as well as handler errors when the API promises a common error schema.
- Choose middleware ordering deliberately: tracing, request IDs, authentication, body limits, timeouts, load shedding, and concurrency limits interact. Ensure fallible Tower services have errors converted into HTTP responses at the correct boundary. Verify whether a limit applies per service clone or across the intended shared resource.
- Test the constructed Router through the project's service test utilities. Assert status, headers, response body, rejection paths, and relevant middleware behavior; include streaming or disconnect behavior when changed. Connect server graceful shutdown to worker cleanup.

### gRPC with Tonic and Prost

- Treat .proto definitions as public contracts. Preserve package/service/method identities, field numbers, and compatible wire types. Reserve removed field numbers and names; never reuse them. Model presence deliberately so an absent value is distinguishable from its default when required. Handle unknown enum values according to the contract instead of silently treating them as a valid domain state.
- Match Tonic, Prost, codec, and build dependencies to the project's versions. Follow its existing build.rs, protoc, include paths, descriptor sets, and generated-code policy; regenerate from schemas rather than hand-edit generated output. Some versions use tonic-prost and tonic-prost-build while older setups differ: verify the installed version before choosing generation APIs. Ensure code generation is reproducible in CI and cross-compilation uses an appropriate host-side compiler.
- Convert protobuf messages into validated domain values at the boundary. Map failures to appropriate tonic::Status codes with sanitized messages; preserve the distinction between malformed input, missing resources, failed preconditions, authentication, authorization, resource exhaustion, and transient unavailability. gRPC status and trailers carry RPC outcomes; HTTP status alone is insufficient.
- Reuse Channels and cheaply cloned generated clients rather than reconnect per RPC or serialize all calls behind one client mutex. Configure connection and RPC budgets separately. A transport-local timeout need not propagate a grpc-timeout header; use the installed version's request deadline mechanism when propagation is required. Pass the remaining budget to downstream calls rather than restarting it. Retry only repeatable operations within that budget; do not transparently replay a partially consumed stream without a defined resume protocol.
- For client, server, or bidirectional streaming, use bounded buffers and respect backpressure. Define ordering, half-close, end-of-stream, terminal errors, and disconnect behavior. Track producer tasks and stop them on cancellation, receiver drop, or shutdown; releasing a request future does not automatically undo committed effects. Enforce suitable message size, concurrency, and stream lifetime/idle limits on the relevant endpoints.
- Apply TLS/mTLS and metadata authentication as required by the deployment. Use interceptors for suitable metadata checks and Tower/service logic when asynchronous authorization or other work is needed. Bound and sanitize metadata, propagate approved correlation fields, and keep tokens out of spans. Enable health/reflection services only when required and configure their exposure deliberately.
- Test changed RPCs using generated clients against a local test server when transport behavior matters. Assert protobuf values, Status codes, required metadata/trailers, deadline behavior, and stream termination/backpressure as applicable. Include schema compatibility fixtures for changed contracts and verify workers clean up after cancellation and shutdown.

### Serde

- Treat serialized data as a public contract. Make field names, missing versus null, defaults, enum tagging, unknown fields, and numeric representation deliberate. Use deny_unknown_fields only when rejecting extensions is part of the contract; blanket strictness can break compatibility.
- Deserialize boundary DTOs and validate into domain types, or use validated custom deserialization. Deriving Deserialize must not bypass invariants enforced by constructors. Verify feature-dependent representations such as decimal and timestamp formats.
- Add focused round-trip and compatibility fixtures for changed formats. Test old payloads when backward compatibility is promised, and prevent sensitive fields from appearing in serialized output.

### Reqwest

- Reuse a configured Client to share its connection pool. Choose TLS, proxy, redirect, and credential behavior for the actual environment; retain certificate verification. For caller-controlled destinations, validate the destination and redirect policy against the intended trust boundary.
- Set connect and request/deadline budgets appropriate to the operation. Inspect HTTP status explicitly; a completed send does not mean a successful status. Bound actual response bytes while reading, including unknown-length responses, instead of trusting Content-Length alone.
- Retry only appropriate failures and operations that are idempotent or protected by an idempotency protocol. Honor throttling, cap attempts/backoff, and include retries and body reads in the overall budget. Verify whether the installed client already retries before adding another retry layer.
- Test with a local fake server or the project's HTTP test facility, including non-success statuses, malformed bodies, slow responses, and redirects where applicable. Do not log authorization headers, credential-bearing URLs, or sensitive response bodies.

### SQLx

- Reuse a bounded pool with appropriate acquisition timeouts. Bind data values rather than interpolate SQL. For dynamic identifiers or ordering, select from validated allowlists; query parameters cannot represent SQL identifiers.
- Keep related database effects in a transaction and explicitly commit success. An unfinished transaction rolls back on drop; handle explicit commit/rollback errors when relevant. Avoid holding a transaction or connection across unrelated network I/O.
- Confirm affected-row counts, nullability, uniqueness, ordering, and isolation assumptions. Use constraints for database invariants and retry transactions only when their complete effects can safely be repeated.
- Follow the repository's query checking and migration workflow. Compile-time macros need the expected database schema or valid offline metadata; use the pinned tooling and refresh metadata when required. Test migrations and queries against the actual database engine in an isolated environment rather than assuming an in-memory substitute has identical semantics.

### Tracing

- Let the application initialize subscribers and filters; libraries emit events/spans without installing a global subscriber. Follow existing field names and correlation conventions, and control high-cardinality fields.
- Use instrumented futures or the crate's async-aware instrumentation. Do not keep a span enter guard across an await; it can attribute other tasks' events to the wrong span.
- Inspect instrument macro argument capture: skip secrets, request bodies, credentials, and expensive or sensitive Debug values. Record only useful sanitized fields. Avoid duplicate logging of the same propagated error at every layer.

Official starting points, to resolve against the project's versions when needed: [Tokio](https://docs.rs/tokio/), [Axum](https://docs.rs/axum/), [Tower](https://docs.rs/tower/), [Tonic](https://docs.rs/tonic/), [Tonic request deadlines](https://docs.rs/tonic/latest/tonic/struct.Request.html#method.set_timeout), [Prost](https://docs.rs/prost/), [Tonic/Prost code generation](https://docs.rs/tonic-prost-build/), [Protobuf compatibility and presence](https://protobuf.dev/programming-guides/proto3/), [Serde](https://serde.rs/), [Reqwest](https://docs.rs/reqwest/), [SQLx](https://docs.rs/sqlx/), and [Tracing](https://docs.rs/tracing/).

## Verify

Add focused behavioral tests proportional to the change. Exercise relevant boundaries, failure paths, resource cleanup, and compatibility. Use fake external dependencies or controllable clocks when needed. Use property tests, benchmarks, Miri, or target-specific checks when the changed invariant warrants them and the tools are available; do not add them as ceremony.

Run repository checks. If none are defined, use cargo fmt --check, scoped tests then the relevant suite, and cargo clippy --all-targets for the supported target/features. Use --all-features only when features are compatible; verify no-default-features or cross-target configurations when promised by the project. Do not silently change MSRV, enable std, or suppress warnings to pass. Use test resources rather than live production systems.

Report changed behavior, key assumptions, checks actually run, and remaining limitations. Never imply unrun checks passed.
