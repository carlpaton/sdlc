---
issue: <number>
title: <short title>
version: <version, if the project uses one>
status: prepared | deploying | released | rolled-back
---

# Release: <title>

## What changed

In plain language, for the people who use or run this: the outcome, taken from the intent's goal and the requirements satisfied.

## Included

| PR | Purpose | State |
|----|---------|-------|
| #<n> | | merged |

## Deviations

Anything that differs from the spec, with the decision record. Write "none" if nothing does.

-

## Pre-flight

| Item | Status |
|------|--------|
| Verification passed (`verification/<number>-<slug>.md`) | done |

## Rollback

How this release is undone, from the conventions, with anything specific to this release (for example a migration that cannot be reversed).

## Deployment log

One row per environment, in order. Filled in as each completes.

| Environment | Route | Approved by | When | Post-deploy checks | Result |
|-------------|-------|-------------|------|--------------------|--------|
| | pipeline / manual | | | | |

## Sources

- Intent: `intent/<number>-<slug>.md`
- Verification: `verification/<number>-<slug>.md`
- Issue: #<number>
