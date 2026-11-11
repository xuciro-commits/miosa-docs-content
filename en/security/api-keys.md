---
slug: "security/api-keys"
title: "API keys"
description: "Create, scope, rotate and revoke the keys programs use, with the presets and limits the platform enforces."
section: "Security & compliance"
order: 14001
---

A key authenticates a program. It carries a list of scopes, an optional workspace
pin, an optional expiry, an optional source-IP allowlist and a rate limit, and it is
shown once, when it is created.

## Everyday commands

```bash
miosa api-key list                      # revoked keys hidden unless --all
miosa api-key create pipeline \
  --preset ci --expires-in 90 --rate-limit 600 \
  --allowed-ip 203.0.113.0/24
miosa api-key get pipeline              # never prints the secret
miosa api-key rotate pipeline           # old secret stops working at once
miosa api-key rm pipeline               # revoke
miosa api-key events pipeline           # when it was used, what changed
```

`--quiet-key` prints only the secret, for a script that pipes it somewhere safe.
A secret cannot be recovered later; rotate to get a new one.

## Scopes

`--scope` takes one scope per flag and can be repeated. A few worth knowing:

| Scope | What it allows |
|---|---|
| `projects:read` / `projects:write` | Projects, Deploy targets, services, releases, preview settings |
| `env:read` / `env:write` / `env:reveal` | Variables, writing them, and reading a value back |
| `sandboxes:read` / `sandboxes:write` / `sandboxes:exec` | Sandbox lifecycle and commands |
| `databases:read` / `databases:write` | Managed databases, including the public endpoint |
| `deployments:read` / `deployments:write` | Docker Host deployments |
| `previews:read` / `previews:write` | Sandbox previews, not pull-request previews |

Pull-request preview settings are `projects:read` / `projects:write`: the sandbox
`previews:*` scopes never grant them.

## Presets

`miosa api-key presets` lists what the server offers; `--preset` picks one at
create time. The named ones are `read-only`, `ci`, `agent`, `cli`, `automation`
and `full-access`. A preset is a starting point, not a ceiling: add `--scope` for
anything it does not include, and the created key lists exactly what it holds.

For one end user of your product, `miosa api-key scoped` mints a key bound to that
person instead of the organization.

> **Note: Reveal is its own scope**
>
> `env:reveal` is separate from `env:read` on purpose: a key that lists variables
> cannot read their values. Keep it off keys that do not need it, and never put a
> key in a client-side bundle.
