---
name: tf-blob
description: Add Azure blob storage to an existing Terraform web or worker environment created by tf-web or tf-worker, private, public (anonymous read of blobs) or both. Use when the user wants file or asset storage on an existing service.
argument-hint: [private|public|both]
---

# Terraform add-on: blob storage

Adds a storage account and container to existing environment directories.

## Inputs

`private`, `public` or `both` (ask if not given), and which environments. Public and private are separate storage accounts, because anonymous access is an account-level setting. Warn the user that anything in the public container is readable by anyone with the URL, and that it cannot be listed.

## Add

1. Find the target directories: when called from `/tf-web` or `/tf-worker`, the ones just created, otherwise ask or look for `resource-group.tf` under the layout. Read the `prefix`, `service` and `env` from `resource-group.tf` and the base doc's naming rules in [../_shared/terraform-base.md](../_shared/terraform-base.md). Stop if there is no base to attach to.
2. Per directory and per kind, generate a fresh 12 hex and render:
   - private: [blob-storage.tf.tmpl](blob-storage.tf.tmpl) to `blob-storage.tf` (`{{blob_hex}}`)
   - public: [blob-storage-public.tf.tmpl](blob-storage-public.tf.tmpl) to `blob-storage-public.tf` (`{{blob_public_hex}}`)
3. Check `prefix` plus the hex is at most 24 characters, lowercase.
4. Run the checks from the base doc, and report the account and container names.

Refuse if the target file already exists. Adding the public container changes no existing resource.
