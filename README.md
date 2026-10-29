# SDLC

A reusable, AI-first software development lifecycle (SDLC) template to apply to future projects.

Each step of the lifecycle is driven by [Claude](https://claude.ai/code) tooling (skills, subagents, hooks, MCP and so on, see [Mechanisms](#mechanisms)), so the process can be run consistently from project to project.

## Status

Early stage. The repository is new and the lifecycle steps and the mechanisms behind them have not been defined yet.

## SDLC steps

Draft list to iterate on. Add, remove, reorder or rename freely. Each step is implemented by whichever mechanism fits best, which may be a skill, a subagent, a hook, an MCP server or a mix.

The steps follow Anthropic's AI-native SDLC playbook (see [Sources](#sources)). Each step hands off to the next through a committed markdown file, so every stage is reviewable and machine-readable.

| # | Step | Purpose | Output | Mechanisms |
|---|------|---------|--------|------------|
| 0 | [Setup](.claude/skills/setup/README.md) | Once per project: record how the code will be built (architecture, testing, conventions) for a new or existing code base | `docs/conventions/`, `docs/decisions/`, `CLAUDE.md` | [`/setup`](.claude/skills/setup/README.md) skill, stack profiles |
| 1 | [Plan](.claude/skills/plan/README.md) | Capture the idea, problem and scope, and choose the [fast or full path](#fast-and-full-paths) | `intent/<issue#>-<slug>.md` | Issue form, [`/issue`](.claude/skills/issue/README.md) and [`/plan`](.claude/skills/plan/README.md) skills |
| 2 | [Design](.claude/skills/design/README.md) | Full path only: turn the approved intent into requirements and design, with a clarify pass to resolve ambiguity. Skipped on the fast path. | `spec/<issue#>-<slug>.md`, decision records | [`/design`](.claude/skills/design/README.md) skill |
| 3 | Build | Write and review a plan and task breakdown before coding, then implement against it | `plan.md`, code | _tbd_ |
| 4 | Test | Give Claude a way to verify its own work; run continuous evals in CI | tests, CI evals | _tbd_ |
| 5 | Deploy | Layered AI review of PRs, hooks as approval gates, CI/CD that stops at the production gate | reviewed PR, release | _tbd_ |
| 6 | Maintain | Trigger Claude from monitoring breaches and scheduled scans; write findings back as new intent | `intent.md` (feeds step 1) | _tbd_ |

### Fast and full paths

Not every change needs a separate design. Plan decides which path a piece of work takes and records it as `path` in the intent.

| | Fast | Full |
|---|---|---|
| For | Small work with an obvious outcome | Anything else |
| Steps | 0 once, then 1, 3, 4, 5, 6 | 0 once, then 1, 2, 3, 4, 5, 6 |
| Spec | The intent itself, plus acceptance criteria | `spec/<issue#>-<slug>.md` from Design |

Later steps read the spec if one exists, otherwise the intent. When in doubt Plan chooses full, and the product owner can change the path in the PR. See the [Plan README](.claude/skills/plan/README.md#fast-and-full-paths) for the criteria.

### Notes

- Step 0 sits outside the loop. It runs once per project (and again when the foundations change), while steps 1 to 6 run per intent.
- Maintain closes the loop: its findings become the input to Plan.
- `CLAUDE.md` plays the role of the project "constitution" from spec-driven development, holding the principles and conventions every step follows.
- Open question: whether Test should stay a separate step or fold into Build. Anthropic keeps it separate.

## Mechanisms

Options for implementing a step, drawn from the playbook. Not exhaustive, and a step can combine several.

| Mechanism | Good for |
|-----------|----------|
| `CLAUDE.md` | Project context every session needs: commands, conventions, architecture |
| Skills | Institutional knowledge applied consistently (advisory), such as templates and policies |
| Hooks | Deterministic guardrails that must always hold: protected paths, linters, approval gates |
| Subagents | Scoped helpers with their own context and tool limits, such as a verifier |
| Slash commands | Packaging a repeatable pass, such as the spec pass or a PR comment sweep |
| MCP servers | Connecting external tools: browser or screenshot checks, deploy, status and rollback |
| Plan / auto mode | Plan mode to approve a plan before edits; auto mode for tightly specced, well-tested work |
| `claude -p` / Agent SDK | Non-interactive runs in CI, on a schedule or from a webhook |
| Git worktrees | Isolating parallel sessions |

Rules of thumb from the playbook:

- Use `CLAUDE.md` for context every session needs, a skill for guidance that must be applied consistently, and a prompt for one-offs.
- Use a skill where advisory is enough and a hook where a policy must always hold.
- Use a subagent for a scoped job within one task, and parallel sessions for independent tasks.

## Planned structure

- One or more mechanisms per SDLC step, chosen per step
- Guidance on adopting the template in a new project

## Usage

To be documented once the first skills exist.

## Sources

- [The AI-native SDLC playbook](https://claude.com/resources/articles/the-ai-native-sdlc-playbook) (Anthropic): source of the six stages and the file-based handoffs
- [GitHub Spec Kit](https://github.com/blackcode1/spec-kit): spec-driven development commands (constitution, specify, plan, tasks, implement)
- [Claude Code spec-driven development](https://www.augmentcode.com/guides/claude-code-spec-driven-development) (Augment Code)
- [Spec-driven development no Claude Code](https://horadecodar.com.br/?p=47039) (Hora de Codar, in Portuguese): clarify step before planning
