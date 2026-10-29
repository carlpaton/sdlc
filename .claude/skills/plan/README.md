# Step 1: Plan

GitHub Issues are the intake and the repo holds the artifact. This step uses two skills: `/issue` for intake and `/plan` for refinement.

1. **Intake:** Anyone files an issue, either from the Idea form on GitHub (`.github/ISSUE_TEMPLATE/idea.yml`) or by running `/issue`, which interviews the user, creates the issue in the same format, and can group it with labels, a project and a milestone. Either way the issue holds what they want, why, who it is for, and links.
2. **Pickup:** Run `/plan <issue-number>`. The skill reads the issue and any linked sources.
3. **Clarify:** It asks questions where the request is ambiguous, in the terminal or as an issue comment for the requester.
4. **Size:** One intent covers one deliverable or milestone. Too big gets split, overlapping issues get merged.
5. **Generate:** It writes `intent/<issue#>-<slug>.md` (from [template.md](template.md)) on a branch and opens a PR, then comments the PR link on the issue.
6. **Accept:** The product owner merges to accept or closes the PR to reject. The issue is labelled `accepted` or closed with a reason.

**The file wins.** Once merged, the intent file is authoritative. The issue keeps the original request and the discussion. If the issue changes later, running `/plan <issue-number>` again enters sync mode: it lists what is new or contradicting, and proposes edits as a PR. Nothing is overwritten silently, and if a `spec.md` exists it flags that it may be affected.

**Prerequisite:** the GitHub CLI (`gh`), authenticated. Both skills check for it first (see [../_shared/gh-prereq.md](../_shared/gh-prereq.md)), ask before installing it if missing, and prompt the user to run `gh auth login` if needed.

Possible later tooling, not needed for v1: a GitHub Action that adds a `needs-sync` label when an issue with an intent file is edited or commented on, and a hook that validates intent files have all their sections.

## Files

- [SKILL.md](SKILL.md): the `/plan` skill's instructions for Claude
- [template.md](template.md): the intent file template
- [../issue/README.md](../issue/README.md): the `/issue` skill's docs, with [SKILL.md](../issue/SKILL.md) holding its instructions for Claude
- [../_shared/gh-prereq.md](../_shared/gh-prereq.md): the `gh` prerequisite check shared by both skills
- `.github/ISSUE_TEMPLATE/idea.yml`: the intake form, and the source of truth for the issue format
