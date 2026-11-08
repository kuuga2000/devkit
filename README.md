# Devkit

Reusable agent skills for Rust implementation, code review, and specification review. The skills support libraries, CLIs, services, embedded/no_std, and FFI projects, following each project's toolchain, dependencies, and conventions.

The shared `SKILL.md` format supports Claude Code, GitHub Copilot, and Codex when installed into their skill discovery directories.

## Skills

| Skill | Purpose | Codex | Claude Code |
| --- | --- | --- | --- |
| [rust-implement](skills/rust-implement/SKILL.md) | Implement and verify Rust features and fixes | `$rust-implement` | `/rust-implement` |
| [rust-code-review](skills/rust-code-review/SKILL.md) | Find concrete defects and compatibility regressions | `$rust-code-review` | `/rust-code-review` |
| [rust-review-spec](skills/rust-review-spec/SKILL.md) | Find requirement gaps and assess testability | `$rust-review-spec` | `/rust-review-spec` |

The implementation skill includes conditional guidance for Tokio, Axum/Tower, Tonic/Prost, Serde, Reqwest, SQLx, and Tracing. It does not require those dependencies in every Rust project.

## Installation

### Skills CLI

With Node.js 22.20 or later, run this from your target project:

```bash
npx skills add kuuga2000/devkit
```

Select the desired skills and agents in the interactive prompts. For installation across projects, use:

```bash
npx skills add kuuga2000/devkit -g
```

These commands use the third-party [Skills CLI](https://github.com/vercel-labs/skills). The syntax follows its documentation; installation from this repository has not yet been tested end to end.

### Manual installation

The `skills/` directory contains the distributable folders. Install the desired folders into your agent's skill directory; the repository layout alone does not enable them in an agent session. The examples below assume the destination folders do not already exist. Copy a single named folder instead of `skills/.` to install only one skill.

For project-local Codex use, run from the Devkit root:

```bash
mkdir -p /path/to/your-project/.agents/skills
cp -R skills/. /path/to/your-project/.agents/skills/
```

For project-local Claude Code use:

```bash
mkdir -p /path/to/your-project/.claude/skills
cp -R skills/. /path/to/your-project/.claude/skills/
```

For personal Claude Code use across projects:

```bash
mkdir -p ~/.claude/skills
cp -R skills/. ~/.claude/skills/
```

For project-local GitHub Copilot use, run from the Devkit root:

```bash
mkdir -p /path/to/your-project/.github/skills
cp -R skills/. /path/to/your-project/.github/skills/
```

These copies are standalone installations. Copy updated folders again when upgrading. The `agents/openai.yaml` files provide Codex UI metadata; Claude Code and Copilot use `SKILL.md` directly. See the [Copilot CLI skill documentation](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills) and [Copilot in VS Code documentation](https://code.visualstudio.com/docs/agent-customization/agent-skills) for discovery and configuration details.

### Shared installation for both agents

To keep one copy per project, install into `.agents/skills/` first, then run these commands from the target project root:

```bash
mkdir -p .claude/skills
ln -s ../../.agents/skills/rust-implement .claude/skills/rust-implement
ln -s ../../.agents/skills/rust-code-review .claude/skills/rust-code-review
ln -s ../../.agents/skills/rust-review-spec .claude/skills/rust-review-spec
```

This is an alternative to copying into `.claude/skills/`. Where symlinks are unavailable, use separate copies. Claude Code supports project skills and symlinked skill directories as described in its [skills documentation](https://code.claude.com/docs/en/skills).

## Usage

Start your agent in the target project after installation. Restart existing sessions if necessary, then invoke a skill with a task:

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

For Copilot, ask it to use a skill by name, for example: `Use rust-code-review to review my local Rust changes.` Copilot can also select installed skills when their descriptions match the task.

See [Rust skill source-selection notes](RUST-SKILLS.md) for the collection sources used to develop these skills.

## Repository layout

```text
skills/
  rust-implement/
    SKILL.md
    agents/openai.yaml
  rust-code-review/
    SKILL.md
    agents/openai.yaml
  rust-review-spec/
    SKILL.md
    agents/openai.yaml
README.md
RUST-SKILLS.md                  Source-selection notes
LICENSE
.gitignore
```

The skills are self-contained and do not load `collection/` at runtime. That ignored directory may exist locally as third-party reference material. Edit the canonical files under `skills/` when maintaining this repository.

## License

Original Devkit material is licensed under the [MIT License](LICENSE). Third-party material under `collection/` retains its upstream licenses and notices; the root license does not replace them. Consult the applicable upstream terms before copying or redistributing that material.
