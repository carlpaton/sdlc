# Step 4: Test

[Build](../build/README.md) proves each task. Test proves the whole: that what was built meets every requirement (or acceptance criterion), with evidence. It reports and raises issues, and it does not fix product code.

## Use

Run `/test <issue-number>` once the parent issue is labelled `built`. The skill:

1. Checks the prerequisites: `gh`, built work, the requirements or acceptance criteria (in the spec on the [full path](../plan/README.md#fast-and-full-paths), or in the intent on the fast path), and `docs/conventions/`. If one is missing it tells you which skill to run first.
2. **Builds a verification matrix:** for each requirement, whether an automated test covers it, a fresh-context verifier can check it, or it is a gap.
3. **Runs the project's checks:** the full build, lint and test commands, recording the commands and results.
4. **Fills gaps with tests:** for a criterion without an automated test that can have one, it writes an acceptance test (tests only, no product code) and opens a PR for review.
5. **Verifies in a fresh context:** a subagent that has not seen the implementation exercises the remaining criteria and returns evidence and a pass, fail or could-not-check verdict for each.
6. Writes `verification/<issue#>-<slug>.md` (from [template.md](template.md)) with the matrix, evidence, commands run and the overall result.
7. **Raises issues** for failures and gaps, after you confirm the list, linked to the parent. They are fixed through `/build`.
8. Opens a PR with the report and comments the result on the parent issue. On a pass the parent is labelled `verified`.

After fixes, run `/test <issue-number>` again. The new report shows what changed since the last run.

## Rules that matter

- **No weakening:** a test or criterion is never edited, deleted or skipped to make a result pass. A wrong criterion goes back to `/plan` or `/design`.
- **Honest results:** a check that could not be run is "could not check", not a pass.
- **Safe by convention:** it respects the conventions' limits on destructive actions, so on an infrastructure project verification means format, validate and plan review, not an apply.
- **The report is a record, not the truth:** the spec or intent stays authoritative, so there is no "file wins" sync here, just re-verification.

## Mechanisms

- **Skill:** `/test` builds the matrix, runs the checks and writes the report.
- **Subagent:** the verifier works in a fresh context, as the playbook suggests, so it does not inherit the builder's assumptions.
- **MCP (optional):** a browser or screenshot tool lets the verifier check UI behaviour.
- **`CLAUDE.md` and conventions:** hold the commands to run, so verification is a single known step.
- **Later, not needed for v1:**
  - CI that runs the project's checks on every PR and posts results, so verification is continuous and not only on request.
  - Hooks that block edits to test files during fix tasks, so a fix cannot weaken the check.
  - Scheduled `claude -p` evals in CI that run when `CLAUDE.md`, skills or hooks change, as the playbook describes, to catch regressions in how the SDLC itself behaves.

## Files

- [SKILL.md](SKILL.md): the skill's instructions for Claude
- [template.md](template.md): the verification report template
- [../_shared/gh-prereq.md](../_shared/gh-prereq.md): the `gh` prerequisite check shared by the skills
