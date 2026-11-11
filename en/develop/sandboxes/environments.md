---
slug: "develop/sandboxes/environments"
title: "Environment variables"
description: "Set sandbox-level env at create time, inherit account-level runtime env by scope and target, and understand precedence, the per-sandbox env API, secret files, and secrets, and how a value reaches exec."
section: "Sandboxes"
order: 2017
---

An environment variable reaches a sandbox from one of several layers: account-level runtime env resolved by scope, values set on the sandbox itself, secrets bound to an opaque placeholder, or `env` passed on a single exec. This page covers what each layer is, how conflicts resolve, and how to read the live set.

## When values are applied

- On create, variables and secrets are applied before the first user command runs.
- On every resume, the sandbox is re-injected with the current set, so a resumed sandbox sees values set after it was created, not only the values it booted with.
- Secret values reach the sandbox only. The API, CLI, and SDKs return the name, metadata, and a short preview, never the plaintext value.
- A snapshot that can be shared or forked across organizations is scrubbed of agent keys and secret values first.

> **Note: Setup scripts**
>
> A setup script passed at create runs once, in the background, after the sandbox is ready.
> It does not delay `ready`.

## Account-level runtime env

Platform owners define environment variables once and have them inherited by sandboxes, computers, and agent runs. Runtime env is encrypted at rest and never returned as plaintext by the API, CLI, or SDKs. Responses include metadata and a short preview only.

MIOSA resolves runtime env in this order. More specific values override broader values with the same name:

| Precedence | Scope | Applies to |
|---:|---|---|
| 1 | Tenant | Every workspace, project, sandbox, computer, and agent under the tenant |
| 2 | Workspace | Every project, sandbox, computer, and agent in that workspace |
| 3 | Project | Sandboxes, computers, deployments, and agent runs attributed to that project |
| 4 | Resource | Per-sandbox or per-computer env set directly on the resource |
| 5 | Run | Env passed directly to a single exec or agent run |

Each runtime env var also has a **target**. At the same scope, a target-specific value overrides one set to `all`:

| Target | Use it when |
|---|---|
| `all` | The value is safe and useful for every runtime surface |
| `sandbox` | The value should only appear in sandbox exec/build/agent sessions |
| `computer` | The value should only appear in desktop computer commands and agents |
| `agent` | The value should be injected into prompt-driven agent runs on sandboxes or computers |
| `deployment` | Production deployment/runtime boot |

Variable names must be uppercase shell-safe names matching `[A-Z][A-Z0-9_]*`.

> **Note: Runtime env is one layer of the agent configuration stack**
>
> Runtime env carries inherited env vars and provider keys by scope and target. An [agent runtime profile](/docs/api-reference/agent-runtime-profiles) owns runtime choice, tools, policy, and non-secret defaults. A [connector](/docs/security/connect-accounts) brokers provider identity. Use runtime env for direct provider env vars, connectors when MIOSA should broker access, and profiles to attach the right defaults.

## Sandbox-level env

Env vars passed at create time are injected into the VM at boot and visible to all processes. They are read-only once the sandbox is running; `env.list()` returns the live set.

**Python**

```python
# Set at create time
sbx = miosa.sandboxes.create(
    env={"DATABASE_URL": "postgres://...", "API_SECRET": "..."},
)

# Read the live env inside a running sandbox
env = sbx.env.list()
print(env)  # {"DATABASE_URL": "postgres://...", ...}
```

**TypeScript**

```typescript
const sbx = await miosa.sandboxes.create({
  env: { DATABASE_URL: "postgres://...", API_SECRET: "..." },
});

const env = await sbx.env.list();
```

**cURL**

```bash
# Pass at create time (in the POST body)

# Read
curl https://api.miosa.ai/api/v1/sandboxes/$SBX/env \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## Per-sandbox env API

The `/api/v1/sandboxes/{id}/env` API manages a key-value store attached to a sandbox. Variables are encrypted at rest and injected into the sandbox process environment on start. The API never returns plaintext values, only key names and metadata.

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/sandboxes/{id}/env` | List all env vars (names only, no values) |
| `PUT` | `/api/v1/sandboxes/{id}/env` | Bulk set or update env vars |
| `DELETE` | `/api/v1/sandboxes/{id}/env/{key}` | Delete a single env var |

`PUT` is a bulk upsert: existing keys are updated, new keys are created, and keys not present in the request are left unchanged.

```bash
curl -X PUT https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/env \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vars": [
      {"key": "DATABASE_URL", "value": "postgres://user:pass@host/db"},
      {"key": "LOG_LEVEL", "value": "info"}
    ]
  }'
```

> **Note: The API reference has the full contract**
>
> See [Sandbox Env API](/docs/api-reference/sandbox-env) for scopes, response shapes, and the error table, and [Runtime Env](/docs/api-reference/runtime-env) for the inherited-env endpoints.

## Secrets

For credentials you do not want inside the VM at all, store them as secrets. Every sandbox using a secret sees an opaque placeholder env var; the real key never lives inside the VM, and the swap happens on egress.

```python
sandbox.secrets.set(
    name="openai_key",
    value="sk-proj-...",
    expose_as_env="OPENAI_API_KEY",
)
```

Inside the sandbox, `os.environ["OPENAI_API_KEY"]` returns an opaque token such as `miosa-tok-7f2a...`. When your code makes an outbound call, the MIOSA proxy on the VM's host sees the placeholder in the `Authorization` header and swaps it with the real key before forwarding. Rotation is instant: every sandbox picks up the new value on its next outbound request, and the placeholder token does not change.

A secret can be scoped at `tenant`, `workspace`, `user`, `external_user`, or `external_workspace`, and the proxy picks the most specific match for the resource making the request.

> **Caution: Where secrets should live**
>
> Do not store long-lived secrets in a sandbox `env` value: those are stored in the sandbox record and visible to any process inside the VM. Pass them per-exec, use the secret store, or use a connector. See [Secrets](/docs/security/secrets) for the full workflow.

## How a value reaches exec

When a command runs, MIOSA resolves the effective environment in layers and hands it to the process:

1. Inherited runtime env for the sandbox's scope and target is materialized at boot.
2. Sandbox-level env (create-time `env` or the per-sandbox env API) overrides an inherited value with the same name.
3. Secrets appear as placeholder env vars exposed via `expose_as_env`.
4. `env` passed to a single exec merges on top for that command only.

```python
result = sbx.exec.run(
    "pytest tests/ -x",
    {"cwd": "/workspace", "env": {"PYTHONPATH": "/workspace"}, "timeout_sec": 120},
)
```

> **Note: Related**
>
> [Create and connect](/docs/develop/sandboxes/create) shows where `env` is passed at create time, and [Connections (OAuth)](/docs/security/connect-accounts) covers brokered provider credentials.
