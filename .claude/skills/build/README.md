# Step 3: Build

Turns a spec (or a fast-path intent) into a reviewed task plan, then implements the tasks one PR at a time. No code is written until a human has approved the plan.

## Use

Run `/build <issue-number>`. The skill works out which mode applies from the repo's state.

**Plan mode** (no `tasks/` file yet):

1. Checks the prerequisites: `gh`, an accepted input (a `ready` spec on the [full path](../plan/README.md#fast-and-full-paths), or an intent with acceptance criteria on the fast path), and `docs/conventions/`. If one is missing it tells you which skill to run first.
2. Reads the spec, conventions and decisions, and surveys existing code with a subagent.
3. Breaks the work into tasks. Each has what to do, the requirements it satisfies, how it is verified, and its dependencies.
4. Checks coverage: every requirement maps to a task and every task to a requirement. It adds no work the spec does not call for.
5. Writes `tasks/<issue#>-<slug>.md` (from [template.md](template.md)), creates one GitHub task issue per task after you confirm the list, and links them to the parent issue.
6. Opens a PR with the plan. A human approves by merging, and the parent issue is labelled `planned`.

**Implement mode** (plan merged): picks the next ready task, or the one you name, and for it:

1. Branches `build/<parent#>-<task#>-<slug>`.
2. Writes the tests the plan names and the code, following the conventions.
3. Runs the build, lint and test commands and fixes failures.
4. Opens a PR that closes the task issue and lists the requirements it satisfies and how it was verified. It never merges. When all tasks are merged the parent is labelled `built`.

If a task turns out to need something the spec does not cover, the skill stops and raises it. It does not diverge silently or widen scope.

## The file wins

Once merged, the plan is authoritative. Running `/build <issue-number>` when the spec, intent or conventions have changed enters sync mode. It lists the tasks that are now wrong, missing or unneeded, proposes the edits as a PR, and flags any task already built that is affected. Task issues are only changed after the plan PR is approved.

## Boundary with Test

Build writes the tests for each task alongside its code, as the conventions describe. [Step 4](../../../README.md#sdlc-steps) covers verifying the work as a whole: acceptance criteria end to end, and continuous checks in CI.

## Mechanisms

- **Skill:** `/build` plans and implements.
- **Subagent:** surveys existing code during planning.
- **Git worktrees:** run independent tasks in parallel without them colliding on files.
- **Plan approval:** the plan PR is the approval gate. Nothing is implemented before it merges.
- **GitHub sub-issues:** tasks hang off the parent issue. This uses `gh api`, which has not been verified against a real repo yet. If it fails, the skill adds a checklist to the parent issue.
- **Later, not needed for v1:**
  - Hooks that run the formatter after edits, block edits to protected paths, and block edits to test files during fix tasks.
  - A verifier subagent that runs the app in a fresh context to check a finished task.
  - Auto mode for routine tasks once specs are tight and tests cover the change.

## Files

- [SKILL.md](SKILL.md): the skill's instructions for Claude
- [template.md](template.md): the plan file template
- [../_shared/gh-prereq.md](../_shared/gh-prereq.md): the `gh` prerequisite check shared by the skills
