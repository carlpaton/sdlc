---
name: tf-web
description: Scaffold Terraform (Azure) for a web app running in a container, for UAT and prod, from a project prefix. Creates the resource group, Log Analytics, Container Apps environment, container registry, externally ingressed container app and state bootstrap doc, then offers optional add-ons (private or public blob storage, Foundry AI models, PostgreSQL). Use when the user wants infrastructure for a new web app or a new environment of one.
argument-hint: [prefix]
---

# Terraform: web app

Create the Terraform for a containerised web app, one directory per environment.

## Steps

1. Read [../_shared/terraform-base.md](../_shared/terraform-base.md) and follow it with `service` = `web`. Check the repo's existing layout and `CLAUDE.md` conventions first.
2. Collect the inputs. The first argument, if given, is the `prefix`.
3. Ask which add-ons to include, all optional:
   - Blob storage: private, public, both or none (`/tf-blob`)
   - Foundry AI models (`/tf-foundry`)
   - PostgreSQL (`/tf-pgsql`)
   For each chosen add-on, ask its own inputs now, as that skill lists them.
4. **Confirm** the plan before writing: the directories, the files, the generated names, and the add-ons.
5. Write the base files for each environment, using this skill's [container_app.tf.tmpl](container_app.tf.tmpl) and appending [vars-extra.tf.tmpl](vars-extra.tf.tmpl) to `vars.tf`.
6. Apply each chosen add-on to each environment by following its skill's "Add" steps, reusing the inputs already collected. The add-ons assume the base files above exist.
7. Write or extend `docs/state-bootstrap.md` and `.gitignore`.
8. Run the checks from the base doc and report.

## Notes

- The ingress is external on port 8080 (`ingress_target_port`). Change it to the port the app listens on.
- The container runs the Microsoft quickstart image as a placeholder. `ignore_changes` on the image stops Terraform fighting the app pipeline.
- The names the app pipeline hardcodes (resource group, container app, registry) are listed in the final report so they can be recorded as the contract with the application.
