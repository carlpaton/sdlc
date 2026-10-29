# Terraform base (shared by the tf-* skills)

The common path for a container service on Azure. `/tf-web` and `/tf-worker` follow this, and the add-on skills (`/tf-blob`, `/tf-foundry`, `/tf-pgsql`) reuse its inputs, naming and checks.

## Inputs

Ask only for what is missing. Never invent a value, and never put a secret in a file.

| Input | Notes |
|-------|-------|
| `prefix` | Short project prefix, for example `ao`, `resolv`, `onepic`. Lowercase letters and digits, starts with a letter, at most 12 characters (storage account names are `prefix` + 12 hex and Azure caps them at 24). |
| `service` | `web` or `worker`, set by the skill. |
| `envs` | Default `uat` and `prod`. |
| `location` | Default `Australia East`. Derive the CLI form (`australiaeast`) by lowercasing and removing spaces. |
| `created_by` | Development company shortcode for the `created_by` tag. Read it from an existing `vars.tf` in the repo if there is one, otherwise ask. |
| `max_replicas` | Defaults: `1` for uat, `3` for prod. |
| `root` | The repo to write into. Default the current directory. |

Subscription ids are not asked for and are never written to a file. `subscription_id` has no default and is supplied with `TF_VAR_subscription_id` or a git-ignored `terraform.tfvars`.

## Layout

Follow the layout already in the repo. If there is none, use `terraform/<service>/<env>/`, one directory per service and environment with the same file names in each, one resource type per file. Environment differences live in variables with an environment default, not in copied code. If a directory for this service and environment already exists, stop and ask. Never overwrite.

## Naming and tags

- Resources are named `<prefix>-<service>-<env>-<type>`, for example `ao-web-uat-rg`. Types: `rg`, `law`, `cae`, `psql`, `ai`. The container app itself is `<prefix>-<service>-<env>`.
- The container registry name is `<prefix><service>registry<env>` (globally unique, alphanumeric only).
- Terraform resource labels are `<prefix>-<type>`, for example `ao-rg`.
- Globally unique names (storage accounts, backend storage, AI subdomains) are `<prefix>` plus 12 hex characters from a newly generated GUID: `uuidgen | tr -d - | cut -c1-12`. Generate a fresh one for each, and never reuse one across environments.
- Every resource carries the tags `environment`, `created_via` and `created_by` through `local.tags` in `vars.tf`.

## Base files

Render the templates in [base/](base) by replacing the `{{placeholders}}`. Derived values: `{{dir}}` is the environment directory, `{{backend_hex}}` is a fresh 12 hex, `{{location_cli}}` is the CLI form of the location, `{{extra_secret_vars}}` is empty unless an add-on needs a secret variable.

| Template | Writes to |
|----------|-----------|
| `setup.tf.tmpl` | `setup.tf` (provider pinned, `azurerm` backend) |
| `vars.tf.tmpl` | `vars.tf` |
| `resource-group.tf.tmpl` | `resource-group.tf` |
| `log_analytics_workspace.tf.tmpl` | `log_analytics_workspace.tf` |
| `container_app_environment.tf.tmpl` | `container_app_environment.tf` |
| `container-registry.tf.tmpl` | `container-registry.tf` |
| the service's `container_app.tf.tmpl` | `container_app.tf` |
| `state-bootstrap.md.tmpl` | a section of `docs/state-bootstrap.md` |

Also make sure `.gitignore` has the lines in `gitignore.tmpl` (add the missing ones, keep the rest), and that a committed `.terraform.lock.hcl` is not ignored.

A skill's `vars-extra.tf.tmpl` is appended to `vars.tf`.

## Checks

After writing, for each environment directory:

1. `terraform -chdir=<dir> fmt` then `fmt -check`
2. `terraform -chdir=<dir> init -backend=false` then `validate`. This writes `.terraform.lock.hcl`, which is committed.

If `terraform` is missing, say so and skip. Report any failure with its output.

## Rules

- Write files only. Do not commit unless asked.
- Never run `terraform plan` against real state, `apply`, `destroy`, or any `az` command that creates or changes resources. The backend is created by hand from `docs/state-bootstrap.md`, and a person plans and applies.
- Do not put passwords, subscription ids or keys in any file, including `default` values.
- Report what was created, the generated hex names, what the person must do next (bootstrap the backend, supply variables, plan UAT first), and which choices are placeholders (quickstart image, ingress port, SKUs).
