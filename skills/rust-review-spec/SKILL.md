---
name: rust-review-spec
description: Review specifications for Rust projects for ambiguity, contradictions, missing failure semantics, feasibility constraints, compatibility, and testable acceptance criteria. Use for requirements review rather than implementation review.
---

# Review a Rust project specification

Read the requested spec, repository instructions, linked contracts, and relevant existing behavior. Identify users, intended project type, scope, target environments, and promises. If no spec is supplied or discoverable, request its location or content; a checklist alone is not a completed review. Do not implement code or rewrite requirements unless requested.

Separate explicit requirements, evidence-backed constraints, suggested defaults, and open questions. Apply only relevant dimensions below. Avoid importing a web stack, runtime, database, or domain model into unrelated projects.

## Contract completeness

| Dimension | Decisions that change observable behavior or feasibility |
| --- | --- |
| Behavior | Inputs/outputs, invariants, state transitions, limits, invalid and missing input, partial failure? |
| Public interface | Library API, CLI flags/exit codes/stdout/stderr, HTTP schema/status, file format, protocol, or ABI as applicable? Compatibility/versioning promises? |
| Data | Representation, units, range, precision/tolerance, encoding, ordering, overflow, serialization round trips? |
| Resources | Ownership and lifecycle, cleanup on error, memory/allocation limits, concurrency, cancellation, deadlines, shutdown? |
| Environment | Edition/MSRV constraints, supported OS/architectures, feature combinations, std/no_std, allocator, runtime, hardware/toolchain requirements? |
| Safety/security | Trust boundaries, unsafe/FFI caller obligations, input limits, permissions, secrets, required access controls? |
| External effects, if present | File integrity, transactions, retry/idempotency, dependency outage, persistence/recovery, cache freshness and consistency? |
| Quality goals | Measurable latency/throughput/memory/size/real-time targets and measurement conditions? Verification on host versus actual target? |

For library specs, distinguish behavior contracts from unnecessary implementation prescriptions. For CLI tools, examine scripting compatibility and failed/interrupted writes. For embedded/FFI, examine hardware assumptions, ABI/ownership, interrupts, allocation, timing, and testability. For services or financial systems, examine failure budgets, freshness, ordering, and exact arithmetic when required.

Raise Rust feasibility concerns where requirements conflict with MSRV/features, no_std, unsafe guarantees, memory limits, target support, or dependency capabilities. Keep implementation preferences separate from actual contract gaps.

## Acceptance criteria

Check that observable tests can be written without guessing business rules. Map significant requirements to input, preconditions, expected result/effect, and relevant boundary or failure case. Consider empty/minimum/maximum/invalid values, failure during multi-step work, cleanup, compatibility, and concurrency only where applicable.

Identify a realistic verification method for requirements that unit tests cannot establish, such as real-time hardware behavior, ABI interoperability, or performance under a defined workload. Label invented values and proposed defaults as proposals, never approved requirements.

## Deliver

State whether the spec is ready to implement, ready with explicit assumptions, or blocked by named decisions. Lead with material contradictions and missing rules. For each issue cite the section, explain plausible interpretations and their practical difference, and propose a precise clarification. Separate non-blocking improvements.

Include a compact requirement-to-acceptance-criterion mapping and unresolved decisions with implementation impact. Ask only questions material to the scoped work. Require formal approval only if the user or repository requires it. If asked to revise the spec, preserve scope and distinguish resolved changes from pending decisions.
