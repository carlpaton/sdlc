---
name: test
description: Step 4 of the SDLC. Verify that a built piece of work meets its requirements or acceptance criteria as a whole, by running the project's checks, having a fresh-context verifier exercise the behaviour, and recording the evidence in verification/<issue#>-<slug>.md. Adds missing acceptance tests (tests only) and raises issues for failures. Use when given an issue number whose tasks are built, or to re-verify after fixes.
argument-hint: <issue-number>
---

# Test

Build proves each task. This step proves the whole: that what was built meets every requirement (or acceptance criterion), with evidence. It reports and raises issues. It does not fix product code, and it never edits a test or criterion to make something pass.

## Prerequisites

Follow [../_shared/gh-prereq.md](../_shared/gh-prereq.md) first. Then check, and stop with a clear message if any fails:

1. The work is built: the parent issue is labelled `built`, or every task PR in `tasks/<issue#>-*.md` is merged. If not, tell the user to finish `/build <issue#>`.
2. The criteria exist. Read `intent/<issue#>-*.md` and its `path`:
   - `path: full`: the requirements and acceptance criteria are in `spec/<issue#>-*.md`.
   - `path: fast`: they are the acceptance criteria in the intent.
3. `docs/conventions/` exists, in particular `testing.md` for how to run checks. If not, tell the user to run `/setup`.

## Mode

Look for `verification/<issue#>-*.md`. If it exists, this is a **re-verification**: read the previous report, and in the new one show what changed. The report is a record of a run, not a source of truth. The spec or intent stays authoritative.

## Steps

1. **Build the verification matrix.** List every requirement (or acceptance criterion). For each, find how it is checked:
   - an existing automated test (name it), or
   - a check a fresh-context verifier can run (a command, a request, a browser or screenshot check through an MCP tool, a query), or
   - neither, which is a gap.
2. **Run the project's checks.** The full build, lint and test commands from `CLAUDE.md` and `docs/conventions/testing.md`. Record the exact commands and results. Respect the conventions' limits on destructive actions (for example, never apply infrastructure changes or run against a shared database without explicit approval).
3. **Fill the gaps with tests.** For a criterion that has no automated test but can have one, write an acceptance test following the conventions, in a branch `test/<issue#>-<slug>`. Write tests only, no product code. Show the user the new tests and open a PR that references the parent issue. Do not merge it.
4. **Verify in a fresh context.** For criteria that automated tests do not cover, use a subagent that has not seen the implementation work. Give it only the criteria and how to run the thing. Ask it for evidence per criterion (output, response, screenshot), and a pass, fail or could-not-check verdict. Do not tell it what the answer should be.
5. **Write the report** to `verification/<issue#>-<slug>.md` using [template.md](template.md): the matrix with a result and evidence for each criterion, the commands run, the gaps, and an overall result.
6. **Raise issues for failures and gaps.** Confirm the list with the user first. For each failing criterion create an issue with `gh issue create` (the `bug` label, creating it if missing, plus the parent's milestone and project), with the criterion, the evidence and how to reproduce, linked to the parent as a sub-issue (see `/build` for the `gh api` approach and checklist fallback). Do not fix them here. They are fixed through `/build`.
7. **Open the PR** with the report on a branch `verification/<issue#>-<slug>`, and comment the result on the parent issue.

On an overall pass, label the parent issue `verified`, creating the label first if it does not exist. On a fail or an unresolved gap, do not label it. Say what blocks it.

## Rules

- Never edit, delete or skip a test, or reword a criterion, to make a result pass. If a criterion is wrong, raise it for `/plan` or `/design`.
- Report results honestly, with output as evidence. A check that could not be run is "could not check", not a pass.
- The only code this step writes is acceptance tests. Anything else is an issue.
- Do not mention a specific project in this skill or template. Project detail belongs in the verification files.
