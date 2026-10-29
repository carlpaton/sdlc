---
name: tf-foundry
description: Add an Azure AI (OpenAI or AI Services) account with model deployments to an existing Terraform web or worker environment created by tf-web or tf-worker. Models are a variable, so adding one is a one-line change. Use when the user wants AI models such as chat or embeddings on an existing service.
argument-hint: [chat model] [embedding model]
---

# Terraform add-on: Foundry AI models

Adds a cognitive account and a map of model deployments to existing environment directories.

## Inputs

| Input | Notes |
|-------|-------|
| kind | `OpenAI` (default) or `AIServices` for non-OpenAI models |
| models | Deployment name, model name, version, SKU and capacity for each. Default to a chat model and an embedding model if the user does not say. Take the current values from the user, or from an existing repo as a reference, such as `gpt-5` `2025-08-07` and `text-embedding-3-small` `1`, both `GlobalStandard` capacity `10`. Model versions and regional availability change, so tell the user to confirm them against their subscription and region before planning. |
| environments | Models may differ per environment, for example a cheaper capacity in uat. |

## Add

1. Find the target directories and read `prefix`, `service` and `env`, as in [../tf-blob/SKILL.md](../tf-blob/SKILL.md) step 1. Stop if there is no base to attach to.
2. Per directory, generate a fresh 12 hex for `{{foundry_hex}}`, set `{{foundry_kind}}`, and build `{{model_entries}}` as one HCL object per deployment:

   ```
       chat = { format = "OpenAI", model = "gpt-5", version = "2025-08-07", sku = "GlobalStandard", capacity = 10 }
   ```

   Render [foundry.tf.tmpl](foundry.tf.tmpl) to `foundry.tf` and append [vars-extra.tf.tmpl](vars-extra.tf.tmpl) to `vars.tf`.
3. Run the checks from the base doc. Report the deployment names and the outputs (`ai_endpoint`, `ai_deployments`).

## Notes

- No keys are output or stored. The app should authenticate with a managed identity or read a key at deploy time from the portal or CLI, recorded in a runbook.
- Capacity is a quota. A plan can succeed while the apply fails for lack of quota, so say so in the report.
