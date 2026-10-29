# Profile: Infrastructure

Questions to ask and defaults to offer when setting up a repo that holds infrastructure as code (Terraform) and database scripts (SQL). Offer the default, record the user's choice.

Verification stands in for testing here. Record it in `docs/conventions/testing.md`.

## Architecture

Ask:

1. **What is managed.** Which cloud provider, which resource types, and which applications they support.
2. **Environments.** Which exist (for example UAT and production), and does an environment span several regions?
3. **Layout.** How is the code split: by service, by environment, by both? Are modules shared, or is each environment self-contained?
4. **State.** Where does Terraform state live, how is that location created, and is there one state per service and environment?
5. **Database scope.** Which engines and databases, and does the data model use one schema per tenant, per customer or per module?

Defaults:

- Terraform, with provider versions pinned and `.terraform.lock.hcl` committed.
- One directory per service and environment (`terraform/<service>/<env>/`), with the same file names in each, one resource type per file. Extract a module once the same block is copied across several environments.
- Remote state, one state per service and environment, in storage that is created outside Terraform (it cannot create its own backend). Record how that was bootstrapped.
- Environment differences live in variables, not copied code.
- SQL kept in its own directory, separate from Terraform.

## Testing

There is no unit test suite. Verification replaces it. Ask:

1. **Checks.** Which run before a change is accepted: format, validate, lint, plan, policy scan?
2. **Plan review.** Who reads a plan before it is applied, and what are they looking for (unexpected destroys, changes to shared resources)?
3. **SQL.** How are scripts proven to work: run against a disposable local database, or only against a real environment?
4. **Where checks run.** On a developer machine, in CI, or both?

Defaults:

- `terraform fmt -check`, `terraform validate` and `terraform plan` on every change, run from the environment directory.
- A plan is read in full before apply. Any destroy or replace is called out and justified.
- SQL scripts run against a throwaway local database (for example a container) before they touch a shared environment.
- Run one environment's checks with `terraform -chdir=<dir> validate`.

## Conventions

Ask:

1. **Who applies.** From a developer machine, or from CI? Is `-auto-approve` allowed?
2. **Authentication.** How do people authenticate to the cloud, and how does the deployment pipeline (a service principal or federated identity)? What scope does each identity have?
3. **Secrets and variables.** Where do they live, how are CI secrets named, and how are they rotated? Nothing secret is committed.
4. **Naming and tagging.** The resource name pattern, and the required tags.
5. **Contract with applications.** Which names (resource groups, app and environment names) do application deploy pipelines hardcode, so that renaming them here breaks a deploy?
6. **Database changes.** How are schema changes numbered and applied, and who may run them against a shared environment?
7. **Runbooks.** Which operational topics need their own doc (DNS, identity, secrets, database)?

Defaults:

- Deployment identities get the least scope that works, such as contributor on a single resource group, one identity per service and environment.
- CI secrets named `<ENV>_<PURPOSE>`, stored in the CI secret store, with the rotation steps written in a runbook.
- `.gitignore` excludes `.terraform/`, `*.tfstate*` and any `*.tfvars` that hold secrets.
- Required tags on every resource: at least who created it, how (portal or Terraform), and which environment.
- SQL changes are numbered, append-only scripts applied in order. An applied script is never edited. A new change is a new script.
- One doc per operational concern in `docs/`, linked from the README, covering how to do it and why it is done that way.
- Shared resource names that applications depend on are listed in one table, and changing one is treated as a breaking change.
- AI agents may run `terraform plan`, but never `apply` or `destroy`, and never run SQL against a shared environment, without explicit approval for that run.

## Deployment

Record in `docs/conventions/deployment.md`. Ask:

1. **Environments.** Which exist and in what order does a change move through them?
2. **Route.** Is a change applied by a person from a machine, or by a pipeline?
3. **Approval.** Who reads the plan and who applies it, for each environment, and how is production gated?
4. **Pre-flight.** What must be true before an apply: login to the right account or subscription confirmed, plan read in full, state location confirmed, dependent applications informed of any renamed resource?
5. **Post-deploy checks.** How is a change confirmed: a clean follow-up plan with no drift, the resource visible and healthy, dependent applications still deploying?
6. **Rollback.** How is a bad change undone (revert the commit and apply again, restore state), and which changes cannot be reversed (deleted data, recreated resources)?
7. **SQL changes.** Who runs a script against each environment, and in what order?

Defaults:

- Apply to the pre-production environment first, and to production only after it is confirmed.
- A person applies, after reading the full plan. Agents hand over the reviewed plan and the exact steps, and do not apply.
- Confirm the active account or subscription before any plan or apply.
- A follow-up `terraform plan` shows no changes once an apply is done.
- Rollback is reverting the change and applying again. Anything destructive or irreversible is called out in the plan review.

## Operations

Record in `docs/conventions/operations.md`. Ask:

1. **Drift.** How is it detected that real resources no longer match the code: a scheduled plan, or by hand?
2. **Cost.** What is the expected spend, and what deviation should be investigated?
3. **Expiry.** Which secrets, certificates and credentials expire, where is the rotation procedure written, and who is reminded?
4. **Backups.** What is backed up, how often, and how is a restore proved?
5. **Availability and capacity.** What is monitored for the resources managed here?
6. **Severity tiers and ownership.** How are issues graded, and who triages?
7. **Live-system access.** What may be read, and what may never be changed outside a reviewed apply?

Defaults:

- A scheduled `terraform plan` per environment that reports a non-empty plan as drift, read-only.
- Spend compared against the expected monthly cost, with a deviation threshold agreed in advance.
- Each expiring credential has a recorded due date and a written rotation procedure.
- Backups have a periodic restore test, with the result recorded.
- Read access is open to the team. Any change to a live environment goes through a reviewed plan and an approved apply.

## Enforcement ideas

Not applied by `/setup` in v1. Note them in the conventions so they can be added later:

- A permission rule or hook that blocks `terraform apply`, `terraform destroy` and writes to shared databases unless approved.
- A hook that runs `terraform fmt` after edits.
- CI that runs `terraform validate`, a linter such as `tflint`, and a policy scanner, and posts the plan on the PR.
- Secret scanning in CI and as a pre-commit check.
- A CI check that fails a plan containing unexpected destroys.
