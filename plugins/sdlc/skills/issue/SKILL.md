---
name: issue
description: Create a well-formed GitHub issue in the same format as the repo's issue forms, and optionally group it with labels, a project and a milestone, creating any that do not exist yet. Use when the user wants to raise, log or file an idea, request, feature, bug or task as a GitHub issue.
argument-hint: [short description]
---

# Issue

Turn what the user wants into a GitHub issue that matches one of the repo's issue forms, so downstream skills such as `/plan` always receive a consistent structure. Optionally group it with labels, a project and a milestone.

## Prerequisites

Follow [../_shared/gh-prereq.md](../_shared/gh-prereq.md) first. Projects also need the `project` token scope. If a project operation fails for scope, ask the user to run `! gh auth refresh -s project`.

## Steps

1. **Pick the form.** List `.github/ISSUE_TEMPLATE/*.yml`. If there is one obvious match for what the user described, use it. If several fit, ask which. If none exist, ask the user for a title and body and skip the form-based steps. The form's `body` fields (label, description, required), `title` prefix and `labels` are the source of truth. Do not hard-code them here.
2. **Interview for the fields**, in the form's order. Start from what the user has already said and only ask for what is missing. Ask a follow-up where an answer is too thin to act on. Do not invent answers, and do not fill in a required field on the user's behalf.
3. **Group it (optional).** Offer labels, a project and a milestone, and let the user skip any of them. Look up what exists first, then let the user pick or create:
   - Labels: `gh label list`. Create with `gh label create <name> --description ... --color ...`. Always include the form's own labels. If one of them does not exist yet, create it and list it in the confirmation, because `gh issue create --label` fails on a missing label.
   - Milestone: `gh api repos/{owner}/{repo}/milestones` to list, and `gh api repos/{owner}/{repo}/milestones -f title=... -f description=...` to create.
   - Project: `gh project list --owner <owner>` to list, and `gh project create --owner <owner> --title ...` to create.
   Suggest an existing match before creating a new one, to avoid near-duplicates. Do not create or assume anything the user has not agreed to.
4. **Build the issue:**
   - Title: the form's title prefix followed by a short summary.
   - Body: for each field, in the form's order, a `### <label>` heading, a blank line, then the answer, then a blank line. For an optional field with no answer, use `_No response_`. This is how GitHub renders a submitted form, so issues created here and through the web form look identical.
5. **Confirm.** Show the draft title and body, plus every label, project and milestone that will be created or applied, and get a clear yes before doing anything. Creating issues, labels, milestones and projects is visible to others.
6. **Create.** Create any new labels, milestone or project first, then run `gh issue create --title ... --body-file ... --label ... --milestone ... --project ...`. Use `--body-file` to preserve formatting.
7. **Report** the issue number and URL, and what was created along the way. If the issue came from the Idea form, tell the user they can run `/plan <number>` next.

## Rules

- Do not mention a specific project in this skill. Project detail belongs in the issue.
- Capture the request as the user means it. Scoping, sizing and splitting belong to `/plan`.
