# Devkit

Reusable agent skills for Rust implementation, code review, and specification review. The skills support libraries, CLIs, services, embedded/no_std, and FFI projects, following each project's toolchain, dependencies, and conventions.

## Skills

| Skill | Purpose | Codex | Claude Code |
| --- | --- | --- | --- |
| [rust-implement](project-proto/.agents/skills/rust-implement/SKILL.md) | Implement and verify Rust features and fixes | `$rust-implement` | `/rust-implement` |
| [rust-code-review](project-proto/.agents/skills/rust-code-review/SKILL.md) | Find concrete defects and compatibility regressions | `$rust-code-review` | `/rust-code-review` |
| [rust-review-spec](project-proto/.agents/skills/rust-review-spec/SKILL.md) | Find requirement gaps and assess testability | `$rust-review-spec` | `/rust-review-spec` |

The implementation skill includes conditional guidance for Tokio, Axum/Tower, Tonic/Prost, Serde, Reqwest, SQLx, and Tracing. It does not require those dependencies in every Rust project.

## Use in this repository

Start your agent from `project-proto/` so its project-local skills are available. Invoke a skill with a task, for example:

```text
# Codex
$rust-implement implement the requirements in specs/feature.md
$rust-code-review review my local changes
$rust-review-spec review specs/feature.md

# Claude Code
/rust-implement implement the requirements in specs/feature.md
/rust-code-review review my local changes
/rust-review-spec review specs/feature.md
```

The spec path is illustrative; substitute your actual file. The skills guide the agent; they do not run a Rust application or replace project tests and review.

## Reuse in another repository

From the Devkit root, copy the canonical skill folders into your target repository:

```bash
mkdir -p /path/to/your-project/.agents/skills
cp -R project-proto/.agents/skills/. /path/to/your-project/.agents/skills/
```

For Claude Code, create these relative links from your target repository root:

```bash
mkdir -p .claude/skills
ln -s ../../.agents/skills/rust-implement .claude/skills/rust-implement
ln -s ../../.agents/skills/rust-code-review .claude/skills/rust-code-review
ln -s ../../.agents/skills/rust-review-spec .claude/skills/rust-review-spec
```

These commands assume the destination skill folders and links do not already exist. Both agents then read the same instructions. Where symlinks are unavailable, copy the canonical folders directly into `.claude/skills/` and keep those copies synchronized manually. For personal Claude Code use across repositories, copy them into `~/.claude/skills/` instead.

See [Rust skill setup and source-selection notes](project-proto/RUST-SKILLS.md) for more details.

## Repository layout

```text
collection/                    Third-party reference collections
project-proto/
  .agents/skills/               Canonical skill files and Codex UI metadata
  .claude/skills/               Relative symlinks for Claude Code discovery
  RUST-SKILLS.md                Usage and source-selection notes
```

The skills are self-contained and do not load `collection/` at runtime. Edit canonical files under `project-proto/.agents/skills/`; Claude Code links automatically reflect those changes.

## License

Original Devkit material is licensed under the [MIT License](LICENSE). Third-party material under `collection/` retains its upstream licenses and notices; the root license does not replace them. Consult the applicable upstream terms before copying or redistributing that material.
