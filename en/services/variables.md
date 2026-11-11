---
slug: "services/variables"
title: "Services variables"
description: "Set a service's variables, reference another service's, share variables across an environment, and generate secrets."
section: "Services & projects"
order: 1009
---

Variables are stored sealed at rest.
A read returns a masked preview, never the value.
The only plaintext read is `reveal`, which needs `env:reveal` and a person's own credential, and which refuses a sealed variable.

## Set a variable

```bash
miosa service vars set shop API NODE_ENV=production
miosa service vars list shop API
```

TypeScript:

```typescript
await miosa.services.setVariables(serviceId, { NODE_ENV: "production" });
const variables = await miosa.services.variables(serviceId);
```

Python:

```python
miosa.services.set_variables(service_id, {"NODE_ENV": "production"})
variables = miosa.services.variables(service_id)
```

A write merges by name. Deleting takes names:

```bash
miosa service vars unset shop API NODE_ENV,OLD_FLAG
```

## References

A value may be a reference.
MIOSA resolves it when the release starts, so the service always sees the current value.

| Expression | Resolves to |
|---|---|
| `${{Postgres.DATABASE_URL}}` | A variable of another service in the same environment |
| `${{shared.API_BASE}}` | An environment shared variable |
| `${{NAME}}` | Another variable of the same service |
| `${{secret(32)}}` | A generated random secret of 32 characters |

```bash
miosa service vars set shop API DATABASE_URL='${{Postgres.DATABASE_URL}}'
```

Quote the value in a shell so `${{...}}` reaches MIOSA unchanged.

A cycle or a malformed expression is refused with every problem at once, and nothing is written.
A reference to a service or variable that does not exist yet is allowed to be saved, but it blocks the deploy and shows up under `problems`.

## Shared variables

Shared variables belong to the environment and are visible to every service in it.

```bash
miosa project vars set shop API_BASE=https://api.acme.dev
miosa project vars list shop
miosa project vars unset shop API_BASE
```

A service reads one with `${{shared.API_BASE}}`.

## When a change needs a release

Writing a variable does not change the running release.
MIOSA records what the write made stale on the service document as `pending_changes`:

```json
{"needs": "rebuild", "variables": ["DATABASE_URL"], "because": "Web.DATABASE_URL", "at": "..."}
```

- `needs: rebuild` when a build-visible variable changed, because a redeploy alone would ship the value inlined at build time.
- `needs: redeploy` for every other variable, which is read when the release starts.
- `because` is `own`, `shared`, or the reference that carried the change: changing `Web.DATABASE_URL` marks every service whose expression uses it, and changing a shared variable marks every service that reads it.

```bash
miosa service get shop API
```

A deploy clears the mark, and a rebuild is never downgraded by a later runtime change:

```bash
miosa service deploy shop API
```

## Check what a deploy will receive

```bash
miosa service vars effective shop API
```

This shows every variable the release will receive, with references resolved and values still masked, plus a `problems` list.
A release is refused while that list is not empty.

## Read one value

```bash
miosa service vars reveal shop API DATABASE_URL
```

Reveal is for a person acting as themselves.
A sealed variable never reveals, and every reveal is audited with the variable's name and never its value.

## Build-time variables

A variable can be marked build-visible, so the build sees it too.
Platform-provided values are never sent to a build.
Names starting with `MIOSA_` or `RAILPACK_` are reserved.
