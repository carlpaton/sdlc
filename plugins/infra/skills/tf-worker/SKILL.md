---
name: tf-worker
description: Scaffold Terraform (Azure) for a console worker running in a container, for UAT and prod, from a project prefix. Same base as the web app but with no ingress, plus optional add-ons (private or public blob storage, Foundry AI models, PostgreSQL). Use when the user wants infrastructure for a background or console worker.
argument-hint: [prefix]
---

# Terraform: console worker

Create the Terraform for a containerised console worker, one directory per environment. It has the same base as `/tf-web` and differs only in the container app, which has no ingress.

## Steps

1. Read [../_shared/terraform-base.md](../_shared/terraform-base.md) and follow it with `service` = `worker`. Check the repo's existing layout and `CLAUDE.md` conventions first.
2. Collect the inputs. The first argument, if given, is the `prefix`. If a `web` service for the same prefix already exists, reuse its `location` and `created_by`.
3. Ask which add-ons to include, all optional: blob storage (`/tf-blob`), Foundry AI models (`/tf-foundry`), PostgreSQL (`/tf-pgsql`). The worker has its own resource group and PostgreSQL server, so say so if the web service already has one, and ask whether the worker should share the web service's database instead (then skip PostgreSQL here and note the connection string comes from the web service).
4. **Confirm** the plan before writing: the directories, the files, the generated names, and the add-ons.
5. Write the base files for each environment, using this skill's [container_app.tf.tmpl](container_app.tf.tmpl). There is no `vars-extra`.
6. Apply each chosen add-on by following its skill's "Add" steps, reusing the inputs already collected.
7. Write or extend `docs/state-bootstrap.md` and `.gitignore`.
8. Run the checks from the base doc and report.

## Notes

- With no ingress and no probe, Container Apps keeps the replica running and restarts it on exit. For a worker that should run on a schedule or per message instead, a Container Apps job or a scale rule is the better fit. Say so if the user describes that.
- The container runs the Microsoft quickstart image as a placeholder, and `ignore_changes` on the image stops Terraform fighting the app pipeline.
