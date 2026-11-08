# Reusable Rust skills

Three self-contained skills live under `.agents/skills/` in this prototype. Claude Code discovers the same files through relative directory symlinks in `.claude/skills/`. They apply across Rust libraries, binaries, CLIs, services, embedded/no_std, and FFI projects. Each discovers the actual requirements, target, toolchain, and conventions. They have no runtime dependency on the collection.

| Task | Codex invocation | Claude Code invocation |
| --- | --- | --- |
| Implement or fix | `$rust-implement implement the behavior in specs/feature.md` | `/rust-implement implement the behavior in specs/feature.md` |
| Review code | `$rust-code-review review my local changes` | `/rust-code-review review my local changes` |
| Review requirements | `$rust-review-spec review specs/feature.md` | `/rust-review-spec review specs/feature.md` |

Paths in these examples are illustrative. Start Claude Code from `project-proto/` to load these project skills at startup. For another repository, copy both `.agents/skills/` and `.claude/skills/` while preserving the symlinks and their relative layout. If symlinks are unavailable, copy the desired canonical skill folders directly into that repository's `.claude/skills/`; those independent copies must be kept in sync manually.

For Claude Code availability across all your repositories, copy the desired canonical skill folders into `~/.claude/skills/` instead. This project setup does not install anything into your home directory. The `agents/openai.yaml` files provide Codex UI metadata; Claude Code uses `SKILL.md` and needs no OpenAI metadata. The shared name/description frontmatter supports automatic selection and explicit invocation without extra permissions or tool pre-approval.

Discovery and symlink support follow the [Claude Code skills documentation](https://code.claude.com/docs/en/skills). Existing sessions may need to be restarted before the slash commands appear, depending on the installed CLI version.

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
