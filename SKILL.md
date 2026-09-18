---
name: sops-age
description: Configure and use Git-versioned files encrypted with SOPS and age. Use when secret-bearing files must live in Git as ciphertext, when a repository already uses SOPS age files, or when its age recipients change. Ask before introducing this workflow to a repository.
compatibility: Requires SOPS and age.
---

# SOPS + age

## Perimeter

Use this skill when a file needs both encryption and Git history, such as a deployment configuration or credentials file. If it does not need Git history, keep it in the secret provider instead.

A new setup begins only after the user approves the encrypted paths, recipient ownership, and key provider.

## Milestones

### Approved boundary

The files requiring encryption, their ciphertext names, the age recipients, and the private-identity provider are explicit and consistent with existing repository conventions.

**Complete when:** the user has approved that boundary.

### Proven encryption path

The private identity stays outside the repository and agent output. Only its public recipient appears in `.sops.yaml`:

```yaml
creation_rules:
  - path_regex: ^secrets/.*\.sops\.yaml$
    age: age1replace_with_public_recipient
```

Plaintext and private-key paths are ignored; the intended ciphertext paths are allowed.

A provider supplies the identity only while SOPS runs. For example:

```dotenv
# .sops.env
SOPS_AGE_KEY=op://<vault>/<item>/<field>
```

```sh
op run --env-file=.sops.env -- sops edit secrets/check.sops.yaml
sops filestatus secrets/check.sops.yaml
op run --env-file=.sops.env -- sops decrypt secrets/check.sops.yaml > /dev/null
```

**Complete when:** `filestatus` reports `encrypted: true`, decryption succeeds without printing plaintext, and the repository contains neither the private identity nor plaintext secrets.

### Commit-ready ciphertext

Changes stay behind SOPS:

```sh
sops edit secrets/app.sops.yaml
sops exec-env secrets/app.sops.yaml '<command>'
sops filestatus secrets/app.sops.yaml
git add secrets/app.sops.yaml
git diff --quiet -- secrets/app.sops.yaml
```

Identity-requiring SOPS commands execute under the provider.

**Complete when:** the requested change exists only as ciphertext, `filestatus` reports `encrypted: true`, and the staged file matches the verified worktree file.

A recipient change is complete when `.sops.yaml` and every encrypted file agree after `sops updatekeys`. Revoking an identity also requires rotating the data key and underlying secrets because old Git history remains decryptable by that identity.
