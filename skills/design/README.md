# Step 2: Design

Turns an accepted intent into a spec: what must be true (requirements and acceptance criteria) and how it will be built (design). Requirements and design are written together in one pass, constrained by the conventions from [step 0](../setup/README.md).

## Use

Run `/design <issue-number>`. The skill:

1. Checks the prerequisites: `gh`, an accepted intent file (from [step 1](../plan/README.md)), and `docs/conventions/` (from step 0). If one is missing it tells you which skill to run first. If the intent is on the [fast path](../plan/README.md#fast-and-full-paths), the intent is already the spec, so it says design is not needed and stops, unless you want a spec anyway.
2. Reads the intent, the issue comments, the conventions and decision records, and any policy skills or documents the project lists in `CLAUDE.md`. For an existing code base it uses a subagent to survey the code the intent touches.
3. **Clarifies:** asks questions where requirements or design are ambiguous, in the terminal or as an issue comment for the requester. Unresolved questions go into the spec as blocking or non-blocking.
4. **Specifies:** writes numbered, testable requirements with acceptance criteria, then the design. Where the design deviates from a convention it proposes a decision record with the reason.
5. **Checks traceability:** every in-scope intent item maps to a requirement and back.
6. Writes `spec/<issue#>-<slug>.md` (from [template.md](template.md)) on a branch, opens a PR, and comments the link on the issue.
7. **Review:** the product owner reviews the requirements and a technical reviewer reviews the design. A spec with a blocking open question stays `draft` and must not be merged. After a merge the issue is labelled `designed`. A closed PR is explained on the issue.

## The file wins

Once merged, the spec is authoritative. Running `/design <issue-number>` again enters sync mode. It compares the spec with its inputs (the intent, issue comments since it was written, and the conventions and decisions) and lists what is new or contradicting. Nothing is overwritten silently, and it flags when step 3 work may be affected.

## Scope

Design specifies the work. It does not break it into tasks or write code, which is step 3, and it does not widen the intent's scope. A requirement that seems missing from the intent goes back as an open question for `/plan`.

## Mechanisms

- **Skill:** `/design` does the interview and writes the files.
- **Subagent:** surveys an existing code base so the reading stays out of the session.
- **Policy skills:** security, compliance, brand or UX policy that a project keeps as its own skills are read during Design, and conflicts are flagged. This template ships none, because they belong to the organization.
- **Later, not needed for v1:**
  - A GitHub Action that runs the design pass non-interactively when an intent merges and opens the spec PR, as the playbook describes.
  - A hook or CI check that validates the spec has all its sections and no blocking open questions.
  - Claude Design or similar for UI mockups, linked from the spec.

## Files

- [SKILL.md](SKILL.md): the skill's instructions for Claude
- [template.md](template.md): the spec file template
- [../_shared/gh-prereq.md](../_shared/gh-prereq.md): the `gh` prerequisite check shared by the skills
