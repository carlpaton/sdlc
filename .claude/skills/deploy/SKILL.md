---
name: deploy
description: Step 5 of the SDLC. Check that verified work is ready to release (PRs reviewed and merged, checks green, pre-flight list done), write the release record and notes, and take it through the project's deployment procedure with an explicit human approval at every environment, production included. Use when given an issue number whose work is verified, or to record the outcome of a release.
argument-hint: <issue-number>
---

# Deploy

Get verified work released safely. This skill checks readiness, prepares the release, and walks the project's own deployment procedure. It never deploys, applies or runs anything against an environment without the user's explicit approval for that run, and it never bypasses a gate.

## Prerequisites

Follow [../_shared/gh-prereq.md](../_shared/gh-prereq.md) first. Then check, and stop with a clear message if any fails:

1. The parent issue is labelled `verified`, meaning step 4 passed. If not, tell the user to run `/test <issue#>`.
2. `docs/conventions/deployment.md` exists: the environments and their order, how a release is made, who approves each environment, the post-deploy checks, and the rollback. If not, tell the user to run `/setup` to add it. Do not guess how this project deploys.

## Steps

1. **Check review state.** List every PR tied to the parent issue (plan, tasks, tests, verification). For each, report whether it is merged, approved or open, whether checks are green, and whether review comments are unresolved. Any PR that has not had a review gets one: follow `REVIEW.md` if the repo has it, and use the `/code-review` skill. Do not merge PRs. That is for the humans.
2. **Pre-flight.** Work through the pre-flight list in `docs/conventions/deployment.md`. Typical items are the verification report passing, migrations or scripts prepared, secrets and variables in place, and the rollback understood. Report each as done, not done or not applicable. Stop if a required item is not done.
3. **Prepare the release.** Write `releases/<issue#>-<slug>.md` using [template.md](template.md): what changed in plain language (from the intent's goal and the requirements satisfied), the PRs included, any deviations from the spec, and the deployment and rollback steps for this release. Make the version bump and changelog entry the conventions call for. Open a PR with these on a branch `release/<issue#>-<slug>` and comment the link on the issue. A human merges it.
4. **Deploy, one environment at a time, in the order the conventions give.** Before each one:
   - Say exactly what will happen, where, and how it can be undone.
   - Ask for explicit approval for that environment and that run. An approval for one environment does not cover the next.
   - Production also needs the approval gate the conventions name (for example required reviewers on a CI environment). Do not work around it.

   Then take the route the conventions give:
   - **Pipeline:** trigger or confirm the pipeline run (for example `gh workflow run` or the merge that starts it) and follow it to completion.
   - **Manual:** where the conventions say a person applies the change, give the exact steps and wait for the user to confirm they ran them. Do not run them yourself.
5. **Check after each deploy.** Run the post-deploy checks from the conventions and record the evidence. If a check fails, stop, say so plainly, and propose the rollback from the conventions. Do not roll back without approval.
6. **Record and close.** Update the release record with what was deployed where, when, who approved, and the check results, and commit it through a PR. When production is done, label the parent issue `released`, creating the label first if it does not exist, and ask the user whether to close the issue.

## Rules

- No deploy, apply, migration or destructive command runs without explicit approval for that run. This holds even if the conventions allow automation, because the approval is part of the gate.
- Never merge your own PRs, bypass a failing check, or skip an environment in the conventions' order.
- Follow the conventions' limits on destructive actions. For infrastructure, a plan is reviewed in full, and the change is applied by the person the conventions name.
- Report outcomes honestly, with evidence. A check that could not be run is not a pass.
- Do not mention a specific project in this skill or template. Project detail belongs in the release records.
