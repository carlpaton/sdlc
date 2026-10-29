# Infra

Claude Code skills that scaffold Azure Terraform for the two kinds of app we build: a web app in a container and a console worker in a container. Both share one base, and optional add-ons bolt on blob storage, Foundry AI models and PostgreSQL.

## Installation

```
claude plugin marketplace add carlpaton/skills
claude plugin install infra@carlpaton
```

Skills are then `/infra:tf-web` and so on.

## Skills

| Skill | Use it to |
|-------|-----------|
| `/tf-web [prefix]` | Create a web app: resource group, Log Analytics, Container Apps environment, registry, container app with external ingress |
| `/tf-worker [prefix]` | Create a console worker: the same base, no ingress |
| `/tf-blob` | Add a private and/or public blob storage account to an existing service |
| `/tf-foundry` | Add an AI account and model deployments (a variable map) |
| `/tf-pgsql` | Add a PostgreSQL Flexible Server |

`/tf-web` and `/tf-worker` ask which add-ons you want, so a new service is one command. The add-ons also work on their own against an existing service.

## Inputs

| Input | Default |
|-------|---------|
| prefix (`ao`, `resolv`, `onepic`) | asked, max 12 characters |
| environments | `uat` and `prod` |
| location | `Australia East` |
| `created_by` tag | read from the repo, else asked |

## Output

```
terraform/<service>/<env>/
  setup.tf  vars.tf  resource-group.tf  log_analytics_workspace.tf
  container_app_environment.tf  container-registry.tf  container_app.tf
  [blob-storage.tf  blob-storage-public.tf  foundry.tf  pgsql.tf]
docs/state-bootstrap.md
```

It follows the conventions in `assetoversight.infra`: names are `<prefix>-<service>-<env>-<type>`, every resource has the tags `environment`, `created_via` and `created_by`, state is remote in storage created by hand, and secrets and subscription ids have no defaults.

## Safety

The skills only write files and run `terraform fmt` and `validate` (with `init -backend=false`). They never plan against real state, apply, destroy or change Azure. You create the state storage from `docs/state-bootstrap.md`, then read the plan and apply yourself, UAT first.

## Layout

Shared rules are in [skills/_shared/terraform-base.md](skills/_shared/terraform-base.md) and the base templates in [skills/_shared/base/](skills/_shared/base). Each skill keeps its own templates next to its `SKILL.md`.
