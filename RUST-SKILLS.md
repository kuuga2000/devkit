# Rust skill source-selection notes

The canonical, self-contained skills are in [skills/](skills/). See [README.md](README.md) for installation and invocation. The collection is reference material, not a runtime dependency.

## Collection sources to take

| Source under `collection/` | Contribution |
| --- | --- |
| `claude-skills/skills/rust-engineer` | Main foundation for ownership, traits, errors, async, and Cargo checks |
| `rust-skills/skills/m06-error-handling` | Recoverable failures and propagation |
| `rust-skills/skills/m07-concurrency` and `agents/plugins/systems-programming/skills/rust-async-patterns` | Conditional synchronization, backpressure, cancellation, and task lifecycle guidance |
| `claude-skills/skills/code-reviewer` and `agents/plugins/developer-essentials/skills/code-review-excellence` | Contextual, actionable review and test scrutiny |
| `rsc-harness/skills/specify` | Observable outcomes and acceptance criteria, adapted for reviewing existing specs |
| `rsc-harness/skills/rust` and `rust-skills/skills/domain-web` | Conditional service-specific checks |
| `rust-skills/skills/domain-fintech` | Conditional numeric precision checks for financial projects |

The generated skills synthesize the relevant guidance. They avoid fixed edition/MSRV/framework choices and mandatory approval workflows from the source collections. Domain-specific rules apply only when the actual project requires them. Ownership, performance, and unsafe specialists can provide deeper guidance when a concrete task warrants it.
