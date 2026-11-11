---
slug: "api-reference/runtime-env"
title: "Runtime Env"
description: "Store inherited runtime environment variables for sandboxes, computers, agents, and deployments."
section: "API reference"
order: 12037
---

Runtime Env stores inherited environment variables for tenant, workspace, and
project scopes. Values are encrypted at rest and are never returned in
plaintext. MIOSA materializes them into sandboxes, computers, agents, or
deployments according to the target you choose.

Base path: `/api/v1/runtime-env`

> **Note: Use this for secrets deliberately**
>
> Runtime Env can hold provider keys such as Anthropic, OpenAI, Higgsfield,
> Resend, Stripe, or customer API tokens. Scope them as narrowly as possible and
> avoid putting tenant admin credentials into broad defaults.

## Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/v1/runtime-env` | List inherited env variables |
| `POST` | `/api/v1/runtime-env` | Create or update one variable |
| `GET` | `/api/v1/runtime-env/{id}` | Show one variable without plaintext |
| `DELETE` | `/api/v1/runtime-env/{id}` | Delete one variable |

List supports these filters:

| Query | Values |
|---|---|
| `scope` | `tenant`, `workspace`, `project` |
| `workspace_id` | Workspace id for workspace-scoped variables |
| `project_id` | Project id for project-scoped variables |
| `target` | `all`, `sandbox`, `computer`, `agent`, `deployment` |

## Create

```http
POST /api/v1/runtime-env
Authorization: Bearer msk_u_...
Content-Type: application/json
```

```json
{
  "scope": "workspace",
  "workspace_id": "ws_123",
  "target": "sandbox",
  "name": "ANTHROPIC_API_KEY",
  "value": "sk-ant-...",
  "metadata": {
    "provider": "anthropic",
    "owner": "lumen"
  }
}
```

Environment variable names must be uppercase shell-safe names matching
`/^[A-Z][A-Z0-9_]*$/`.

## Scope model

| Scope | Required id | Use it for |
|---|---|---|
| `tenant` | none | Organization-wide defaults. |
| `workspace` | `workspace_id` | A customer/workspace provider key or tool config. |
| `project` | `project_id` | A single app or deployment pipeline. |

| Target | Receives the variable |
|---|---|
| `all` | All supported runtime targets. |
| `sandbox` | Sandbox sessions and sandbox agent runs. |
| `computer` | Desktop computer sessions and computer agent runs. |
| `agent` | Agent runtime sessions. |
| `deployment` | Production deployment/runtime boot. |

Use narrower scopes for spend-capable keys. For example, set
`ANTHROPIC_API_KEY` at workspace scope for Lumen instead of tenant scope if
that key should only power Lumen sandboxes and computers.

## CLI

Runtime Env has no dedicated `miosa` subcommand; drive it with `miosa api`
against the `/runtime-env` endpoint.

```bash
miosa api POST /runtime-env -d '{
  "name": "ANTHROPIC_API_KEY",
  "value": "sk-ant-...",
  "scope": "workspace",
  "workspace_id": "ws_123",
  "target": "sandbox"
}'

miosa api POST /runtime-env -d '{
  "name": "HIGGSFIELD_API_KEY",
  "value": "hgf_...",
  "scope": "project",
  "project_id": "proj_123",
  "target": "agent"
}'

miosa api GET /runtime-env?scope=workspace&workspace_id=ws_123
miosa api GET /runtime-env/env_123
miosa api DELETE /runtime-env/env_123
```

## Relationship to profiles and connectors

Runtime Env, Agent Runtime Profiles, and managed connectors are separate:

| Primitive | Owns |
|---|---|
| Runtime Env | Inherited environment variables and encrypted provider keys. |
| Agent Runtime Profile | Runtime choice, tools, connectors, default env, and policy. |
| Managed connector | Brokered provider access and scoped token exchange. |

Use Runtime Env for direct provider environment variables. Use connectors when
MIOSA should broker provider access or restrict egress. Use profiles to attach
the right defaults to new sandboxes, computers, and agent runs.

## Next

- [Agent Runtime Profiles](/docs/api-reference/agent-runtime-profiles): Standardize runtimes, tools, connectors, and policy for each workspace or project.
- [Agent Runs](/docs/api-reference/agent-runs): Dispatch prompts with inherited env and runtime profile resolution.
- [Agent orchestration](/docs/guides/agent-orchestration): Put inherited env, profiles, groups, events, and artifacts together.
