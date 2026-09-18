---
name: sops-age
description: Configure and use Git-versioned files encrypted with SOPS and age. Use when secret-bearing files must live in Git as ciphertext, when a repository already uses SOPS age files, or when its age recipients change. Ask before introducing this workflow to a repository.
compatibility: Requires SOPS and age.
---

# SOPS + age

## Perimeter

Use this skill when a file needs both encryption and Git history, such as a deployment configuration or credentials file. If it does not need Git history, keep it in the secret provider instead.

For a new setup, first propose the encrypted paths, recipient ownership, and key provider. Ask the user before creating keys or repository files.

## Workflow

1. **Approve the boundary.** Inspect existing conventions and agree with the user on the plaintext boundary, ciphertext names, recipients, and provider.

   **Milestone:** the user has approved a concrete plan.

2. **Establish and prove encryption.** Have the user store the private age identity outside the repository and agent output. Put only its public recipient in `.sops.yaml`:

   ```yaml
   creation_rules:
     - path_regex: ^secrets/.*\.sops\.yaml$
       age: age1replace_with_public_recipient
   ```

   Ignore private-key and plaintext paths while allowing the approved ciphertext paths.

   The provider only needs to supply the identity while SOPS runs. For example:

   ```dotenv
   # .sops.env
   SOPS_AGE_KEY=op://<vault>/<item>/<field>
   ```

   ```sh
   op run --env-file=.sops.env -- sops edit secrets/check.sops.yaml
   sops filestatus secrets/check.sops.yaml
   op run --env-file=.sops.env -- sops decrypt secrets/check.sops.yaml > /dev/null
   ```

   **Milestone:** `filestatus` reports `encrypted: true`, decryption succeeds without printing plaintext, and no private identity or plaintext secret is in the repository.

3. **Operate and verify.** Edit or pass plaintext to a process through SOPS; never decrypt into the repository.

   ```sh
   sops edit secrets/app.sops.yaml
   sops filestatus secrets/app.sops.yaml
   git add secrets/app.sops.yaml
   git diff --quiet -- secrets/app.sops.yaml
   ```

   Run identity-requiring commands through the provider.

   **Milestone:** the requested change exists only as ciphertext, `filestatus` reports `encrypted: true`, and the staged file matches the verified worktree file.

When recipients change, update `.sops.yaml`, run `sops updatekeys` on every encrypted file, and repeat the relevant milestones. When revoking an identity, also rotate the data key and underlying secrets.
