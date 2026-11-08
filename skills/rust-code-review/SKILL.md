---
name: rust-code-review
description: Review Rust code or diffs across libraries, CLIs, services, embedded, and FFI projects for concrete defects, compatibility regressions, unsafe invariants, and test gaps. Use for code review rather than specification review.
---

# Review Rust code

Review without editing unless fixes are requested. Establish the requested diff/base or file scope. Read repository instructions, requirements, Cargo/toolchain configuration, callers, and relevant tests. With no scope, inspect staged and unstaged changes and state that scope; ask for a target if none exist. Do not silently choose an arbitrary branch base.

Identify project type, target platforms, MSRV, feature combinations, and public contracts before applying checks. Trace changed behavior across boundaries rather than assessing isolated syntax. Compare against actual requirements; missing requirements are questions, not invented defects.

## Find concrete failures

Prioritize relevant checks:

- **Correctness:** invariant violations, boundary cases, overflow/truncation, Unicode/indexing, partial updates, cleanup on errors, and precision or unit errors.
- **Ownership/resources:** unnecessary retention, cycles, leaked handles, expensive cloning on important paths, and lifetime or ownership contracts that break callers.
- **Errors:** input-triggered panics, lost errors, misleading fallbacks, incompatible error variants, and sensitive internal information exposed publicly.
- **Unsafe/FFI:** pointer validity, alignment, initialization, aliasing, provenance, lifetimes, ownership transfer, layout/ABI, unwinding, and thread safety. Verify the invariant behind each safety claim, including safe callers that could violate it.
- **Concurrency when present:** races, deadlocks, lock scope, blocking runtime workers, unbounded tasks/queues, lost task failures, cancellation-induced partial state, retry amplification, and shutdown. A mutex alone does not establish correctness.
- **Compatibility:** public signatures/trait bounds, semver implications, MSRV, supported features/targets, no_std constraints, wire/file formats, and CLI exit/output contracts.
- **Domain boundaries:** HTTP authorization/validation, SQL parameterization, atomic file replacement, cache freshness/order, exact financial arithmetic, or hardware timing only when applicable to the changed component.

Do not impose service, async, finance, or database assumptions on unrelated Rust projects. Identify performance defects through a concrete workload or cost path rather than personal abstraction preferences.

Read tests as behavioral evidence. Check boundaries and failures rather than only mocked internals. Compilation and clippy do not prove semantic correctness. Run focused non-mutating checks when useful with the project's supported target/features. Do not contact live production resources merely to validate a review. Distinguish executed checks, static reasoning, and unresolved hypotheses.

## Report

Lead with actionable findings ordered by impact. Each finding needs severity, a narrow file/line location, triggering input or interleaving, incorrect behavior, user impact, and supporting contract/code evidence. Suggest a focused correction or regression test when useful.

Use P0 for immediate widespread failure, P1 for serious correctness/security/availability failure, P2 for material bugs under specific conditions, and P3 for low-impact defects. Report defensible defects; label uncertain concerns as questions with missing evidence. Keep optional improvements separate and leave formatting to configured tools. Do not invent findings or demand unrelated architectural rewrites.

If no findings exist, say so, then state scope, verification, and material coverage gaps. A clean review does not prove the project bug-free.
