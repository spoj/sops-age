---
name: sops-age
description: Set up and use Git-versioned SOPS files encrypted to age recipients, with private identities supplied at runtime by any provider. Use when configuring, editing, consuming, validating, or rekeying encrypted repository files.
compatibility: Requires SOPS 3.10+ and age.
---

# SOPS + age

- Commit only SOPS ciphertext, `.sops.yaml`, and age public recipients.
- Keep private identities outside the repository. Do not emit identities or plaintext into agent output.
- Supply identities at runtime with `SOPS_AGE_KEY`, `SOPS_AGE_KEY_FILE`, or `SOPS_AGE_KEY_CMD`.

## Configure

Have the user create and store the age identity outside agent-captured output. Use only its public recipient in `.sops.yaml`:

```yaml
creation_rules:
  - path_regex: ^secrets/.*\.sops(\.(yaml|yml|json|env|ini))?$
    age: age1replace_with_public_recipient
```

Ignore plaintext secret paths in Git.

## Use

Run SOPS under the chosen identity provider:

```sh
sops edit secrets/example.sops.yaml
sops exec-env secrets/example.sops.env '<command>'
sops exec-file secrets/example.sops.yaml '<command using {}>'
```

Do not decrypt into the repository. Before committing, check every changed secret with `sops filestatus`. After changing recipients in `.sops.yaml`, run `sops updatekeys` on every encrypted file.

### 1Password example

Store the age identity in 1Password and map it in `.sops.env`:

```dotenv
SOPS_AGE_KEY=op://<vault>/<item>/<field>
```

Then prefix SOPS commands with:

```sh
op run --env-file=.sops.env -- sops ...
```
