# Issue

Creates well-formed GitHub issues, and is the intake for [step 1: Plan](../plan/README.md). It matches the repo's issue forms and can group the issue with labels, a project and a milestone.

## Use

Run `/issue`, optionally with a first description of what you want. The skill:

1. Checks the `gh` prerequisite.
2. Picks the matching issue form from `.github/ISSUE_TEMPLATE/` (today that is Idea).
3. Interviews you for each field, asking only for what is missing and never inventing answers.
4. Offers to group the issue: labels, a project and a milestone. It lists what exists, suggests matches, and creates new ones only if you agree. All three are optional.
5. Shows the draft and everything it will create or apply, and waits for your yes.
6. Creates the issue and reports the number and URL.

For an Idea issue, run `/plan <issue-number>` next to refine it into an intent.

## Why a skill as well as the forms

The issue forms only help people filing through the GitHub web UI. `/issue` covers filing from a Claude session. It writes the body the way GitHub renders a submitted form (`### <label>` headings in the form's order, `_No response_` for empty optional fields), so issues from either route look the same and `/plan` always gets the same structure.

## Grouping

| What | Lists with | Creates with |
|------|-----------|--------------|
| Labels | `gh label list` | `gh label create` |
| Milestone | `gh api repos/{owner}/{repo}/milestones` | the same endpoint with `-f title=...` |
| Project | `gh project list` | `gh project create` |

Projects need the `project` token scope. If it is missing, run `gh auth refresh -s project`.

## Files

- [SKILL.md](SKILL.md): the skill's instructions for Claude
- [../_shared/gh-prereq.md](../_shared/gh-prereq.md): the `gh` prerequisite check shared with `/plan`
- `.github/ISSUE_TEMPLATE/*.yml`: the issue forms, and the source of truth for the fields, title prefix and labels
- `.github/ISSUE_TEMPLATE/config.yml`: disables blank issues so the forms are the only way in. Delete it to allow free-form issues.
