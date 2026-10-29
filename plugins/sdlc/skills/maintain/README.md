# Step 6: Maintain

Closes the loop. A signal from a live system is investigated, triaged and written back as an issue, so fixes follow the same steps as any other work. The skill investigates and records. It does not change a live system.

## Use

Run `/maintain` with an issue number, a pasted alert or scan result, or a description of what you are seeing. The skill:

1. Checks the prerequisites: `gh` and `docs/conventions/operations.md` (from [step 0](../setup/README.md)), which says what is monitored and what counts as normal. If it is missing it tells you to run `/setup`.
2. **Investigates read-only:** gathers evidence from the logs, metrics, scan output and recent releases the conventions point to. It does not restart, change or write to a live system.
3. **States what is known:** impact, likely cause and the evidence, with guesses marked as guesses.
4. **Triages:** proposes a severity and one outcome with reasons. **Fix now** for current or growing impact, **schedule** for real but not urgent, **dismiss** for a non-problem, duplicate or accepted risk. You decide, and a dismissal always records its reason.
5. **Writes it back:** creates an issue with `/issue` (the Idea form, so [`/plan`](../plan/README.md) gets the usual structure) labelled `maintenance`, plus `urgent` for fix now. `urgent` means picked up first, not skipping approvals, review or verification.
6. **Incidents:** after resolution, writes a blameless post-mortem to `incidents/<date>-<slug>.md` (from [template.md](template.md)) and raises each follow-up as an issue.
7. **Learns:** asks whether the cause was a gap the process should have caught, such as a missing convention, test or monitor. If so it proposes the change as a PR, and does not edit conventions without your agreement.

## Rules that matter

- **Read-only on live systems:** any change goes through the loop and [`/deploy`](../deploy/README.md) with its approvals.
- **Honest evidence:** observed facts are kept apart from inferences, and anything that could not be checked is said so.
- **Propose, don't decide:** it never dismisses or closes a signal on your behalf.

## Triggers

Version 1 is run by hand. The playbook describes automating the front of this step, and that is left for later:

- A **deterministic detection script** watches a metric and calls Claude only when a threshold or control band is breached, with the tiers kept in version-controlled config. This keeps cost and noise down.
- A **scheduled workflow, monitoring webhook or cron job** starts a stateless, non-interactive run (`claude -p`). Start read-only, and add write access only once the results are trusted.
- An **Agent SDK service** if something long-running needs to receive webhooks.
- **Scheduled security scans** of the repositories, with findings exported to the issue tracker.
- A **chat integration** that triages incidents in a channel and writes up the post-mortem.

## Mechanisms

- **Skill:** `/maintain` investigates, triages and writes back.
- **`/issue` skill:** turns findings into well-formed issues for Plan.
- **Subagent:** searches logs and code in bulk and returns a summary.
- **MCP (optional):** read-only access to monitoring or logging tools.
- **Conventions:** `docs/conventions/operations.md` holds what is monitored, thresholds, tiers and owners.

## Files

- [SKILL.md](SKILL.md): the skill's instructions for Claude
- [template.md](template.md): the post-mortem template
- [../_shared/gh-prereq.md](../_shared/gh-prereq.md): the `gh` prerequisite check shared by the skills
