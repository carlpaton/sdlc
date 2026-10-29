# Step 5: Deploy

Gets verified work released safely. It checks readiness, prepares the release, and walks the project's own deployment procedure with a human approval at every environment. It does not decide how a project deploys: that comes from `docs/conventions/deployment.md`, written in [step 0](../setup/README.md).

## Use

Run `/deploy <issue-number>` once the parent issue is labelled `verified`. The skill:

1. Checks the prerequisites: `gh`, a `verified` issue, and `docs/conventions/deployment.md`. If one is missing it tells you which skill to run first.
2. **Checks review state:** lists every PR tied to the issue with its merge state, checks and unresolved comments. Any PR that has not been reviewed gets a review with the `/code-review` skill, following `REVIEW.md` if the repo has one. It never merges PRs.
3. **Runs the pre-flight list** from the conventions and stops if a required item is not done.
4. **Prepares the release:** writes `releases/<issue#>-<slug>.md` (from [template.md](template.md)), makes the version bump and changelog entry the conventions call for, and opens a PR for a human to merge.
5. **Deploys one environment at a time**, in the conventions' order. Before each it says what will happen and how to undo it, and asks for approval for that environment and that run. Production also needs the project's own gate, such as required reviewers on a CI environment.
   - **Pipeline:** it triggers or follows the pipeline run.
   - **Manual:** it gives the exact steps and waits for you to confirm you ran them. It does not run them.
6. **Checks after each deploy** with the conventions' post-deploy checks. On a failure it stops and proposes the rollback, and does nothing without approval.
7. Records what was deployed where, when, who approved and the results, labels the issue `released` after production, and asks whether to close it.

## Rules that matter

- **No silent deploys:** nothing runs against an environment without explicit approval for that run, even if the conventions allow automation.
- **Gates stay:** it never bypasses a failing check, skips an environment, or merges its own PRs.
- **Follows the project:** an infrastructure project applies by hand, so the skill hands over the reviewed plan and the steps instead of applying. An application project may deploy by pipeline.
- **Honest results:** a check that could not be run is not a pass.

## Mechanisms

- **Skill:** `/deploy` checks, prepares and walks the procedure.
- **`/code-review` skill and `REVIEW.md`:** review of PRs. `REVIEW.md` defines the review passes, severity and what to skip, and this template does not ship one because it depends on the project.
- **GitHub Environments with required reviewers:** the production gate lives in the pipeline, not only in the skill.
- **Conventions:** `docs/conventions/deployment.md` holds the environments, approvals, post-deploy checks and rollback.
- **Later, not needed for v1:**
  - Automated PR review in CI, through the managed Code Review service or `claude-code-action`, with `@claude` mentions to address comments.
  - Hooks as approval gates, configured in `.claude/settings.json` for team rules and in managed settings for non-negotiable controls.
  - `claude -p` in the pipeline for judgement steps such as build triage, with scoped credentials, starting read-only.
  - MCP servers that expose deploy, status and rollback as scoped tools per environment.
  - Sandboxing and managed settings to limit network access and credentials.

## Files

- [SKILL.md](SKILL.md): the skill's instructions for Claude
- [template.md](template.md): the release record template
- [../_shared/gh-prereq.md](../_shared/gh-prereq.md): the `gh` prerequisite check shared by the skills
