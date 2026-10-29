# SDLC

A reusable, AI-first software development lifecycle (SDLC) template to apply to future projects.

Each step of the lifecycle is driven by [Claude](https://claude.ai/code) tooling (skills, subagents, hooks, MCP and so on, see [Mechanisms](#mechanisms)), so the process can be run consistently from project to project.

## Status

Early stage. Every step (0 to 6) has a first-draft skill, README and template, but none has been run end to end on a real project yet. Expect the skills to change once they are tried. Automation (GitHub Actions, hooks, scheduled runs) is listed as later work in each step's README and is not built.

## SDLC steps

Draft list to iterate on. Add, remove, reorder or rename freely. Each step is implemented by whichever mechanism fits best, which may be a skill, a subagent, a hook, an MCP server or a mix.

The steps follow Anthropic's AI-native SDLC playbook (see [Sources](#sources)). Each step hands off to the next through a committed markdown file, so every stage is reviewable and machine-readable.

| # | Step | Purpose | Output | Mechanisms |
|---|------|---------|--------|------------|
| 0 | [Setup](.claude/skills/setup/README.md) | Once per project: record how the code will be built (architecture, testing, conventions) for a new or existing code base | `docs/conventions/`, `docs/decisions/`, `CLAUDE.md` | [`/setup`](.claude/skills/setup/README.md) skill, stack profiles |
| 1 | [Plan](.claude/skills/plan/README.md) | Capture the idea, problem and scope, and choose the [fast or full path](#fast-and-full-paths) | `intent/<issue#>-<slug>.md` | Issue form, [`/issue`](.claude/skills/issue/README.md) and [`/plan`](.claude/skills/plan/README.md) skills |
| 2 | [Design](.claude/skills/design/README.md) | Full path only: turn the approved intent into requirements and design, with a clarify pass to resolve ambiguity. Skipped on the fast path. | `spec/<issue#>-<slug>.md`, decision records | [`/design`](.claude/skills/design/README.md) skill |
| 3 | [Build](.claude/skills/build/README.md) | Break the spec (or fast-path intent) into tasks and get the plan approved before coding, then implement one task per PR | `tasks/<issue#>-<slug>.md`, task issues, code | [`/build`](.claude/skills/build/README.md) skill |
| 4 | [Test](.claude/skills/test/README.md) | Verify the built work meets every requirement as a whole, with evidence from a fresh-context verifier; add missing acceptance tests and raise issues for failures | `verification/<issue#>-<slug>.md`, acceptance tests, bug issues | [`/test`](.claude/skills/test/README.md) skill |
| 5 | [Deploy](.claude/skills/deploy/README.md) | Check readiness and review state, prepare the release, and walk the project's deployment procedure with explicit human approval at every environment | `releases/<issue#>-<slug>.md`, reviewed PRs, release | [`/deploy`](.claude/skills/deploy/README.md) skill, `/code-review`, `REVIEW.md` |
| 6 | [Maintain](.claude/skills/maintain/README.md) | Investigate signals from live systems read-only, triage them (fix now, schedule, dismiss), write them back as issues for Plan, and learn from incidents | issues (feed step 1), `incidents/<date>-<slug>.md`, convention updates | [`/maintain`](.claude/skills/maintain/README.md) skill, `/issue` |

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
- Test stays separate from Build, as Anthropic keeps it. Build writes each task's tests alongside its code, and Test verifies the whole piece of work against its requirements.

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

## Usage

The skills live in `.claude/skills/`, and each step's README says how it works. Adopting the template in a project means getting those skills and the `.github/ISSUE_TEMPLATE/` forms into that project's repo. How to distribute them (copying, or packaging as a plugin) is not decided yet.

Once per project:

1. `/setup` records the architecture, testing, deployment and operations conventions.

Then for each piece of work, in order:

| Step | Command | Run it when |
|------|---------|-------------|
| 1 Plan | `/issue`, then `/plan <issue>` | Someone has an idea or request |
| 2 Design | `/design <issue>` | The intent is accepted and on the full path |
| 3 Build | `/build <issue>` | The spec (or fast-path intent) is accepted |
| 4 Test | `/test <issue>` | The parent issue is labelled `built` |
| 5 Deploy | `/deploy <issue>` | The parent issue is labelled `verified` |
| 6 Maintain | `/maintain` | A live system raises a signal. Findings come back through step 1. |

Each skill checks its prerequisites and tells you which skill to run if one is missing.

Progress shows up as labels on the parent issue: `accepted`, `designed`, `planned`, `built`, `verified`, `released`.

## Sources

- [The AI-native SDLC playbook](https://claude.com/resources/articles/the-ai-native-sdlc-playbook) (Anthropic): source of the six stages and the file-based handoffs
- [GitHub Spec Kit](https://github.com/blackcode1/spec-kit): spec-driven development commands (constitution, specify, plan, tasks, implement)
- [Claude Code spec-driven development](https://www.augmentcode.com/guides/claude-code-spec-driven-development) (Augment Code)
- [Spec-driven development no Claude Code](https://horadecodar.com.br/?p=47039) (Hora de Codar, in Portuguese): clarify step before planning
