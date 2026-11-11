---
slug: "api-reference/agent-runtime-profiles"
title: "Agent Runtime Profiles"
description: "Configure reusable agent runtime defaults for sandboxes, computers, and custom agent sessions."
section: "API reference"
order: 12036
---

Agent Runtime Profiles define how MIOSA should prepare an agent runtime for a
tenant, workspace, or project. Use profiles to standardize OSA, Codex, Claude
Code, Pi, Hermes, or custom runtimes across sandboxes and computers without
passing the same model, tools, connectors, policy, and environment on every run.

Base path: `/api/v1/agent-runtime-profiles`

> **Note: Profiles are defaults, not prompts**
>
> Profiles configure runtime shape. Agent Runs still provide the prompt, target,
> timeout, and task-specific metadata.

## Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/v1/agent-runtime-profiles` | List profiles for the tenant |
| `POST` | `/api/v1/agent-runtime-profiles` | Create a profile |
| `GET` | `/api/v1/agent-runtime-profiles/{id}` | Fetch one profile |
| `PUT` | `/api/v1/agent-runtime-profiles/{id}` | Update a profile |
| `DELETE` | `/api/v1/agent-runtime-profiles/{id}` | Delete a profile |

List can be filtered with `workspace_id` or `project_id`.

## Create

```http
POST /api/v1/agent-runtime-profiles
Authorization: Bearer msk_u_...
Content-Type: application/json
```

```json
{
  "name": "lumen-claude",
  "runtime": "claude-code",
  "workspace_id": "ws_123",
  "project_id": "proj_123",
  "description": "Default Lumen coding and browser QA runtime",
  "applies_to": {
    "resources": ["sandbox", "computer"]
  },
  "tools": ["filesystem", "shell", "browser", "artifacts"],
  "connectors": ["anthropic/lumen", "refero/managed"],
  "env": {
    "ANTHROPIC_MODEL": "claude-sonnet-4.6"
  },
  "policy": {
    "approval_required_for": ["deploy", "external_write"],
    "max_child_runs": 25
  },
  "metadata": {
    "owner": "platform"
  },
  "is_default": true
}
```

## Fields

| Field | Description |
|---|---|
| `name` | Human-readable profile name. Unique within the relevant tenant/workspace/project scope. |
| `runtime` | One of `osa`, `codex`, `claude`, `claude-code`, `pi`, `hermes`, or `custom`. |
| `workspace_id` | Optional workspace scope. |
| `project_id` | Optional project scope. |
| `description` | Operator-facing explanation. |
| `applies_to` | Resource selector, commonly `{ "resources": ["sandbox", "computer"] }`. |
| `tools` | Tool ids the runtime should expect, such as `filesystem`, `shell`, `browser`, `artifacts`, or `deployments`. |
| `connectors` | Connector UIDs to bind when the target runtime is created. |
| `env` | Non-secret runtime defaults. Put provider keys in runtime env/secrets, not here. |
| `policy` | Product policy such as approvals, max child runs, or external-write rules. |
| `metadata` | Product-specific labels and ids. |
| `is_default` | Whether MIOSA should resolve this profile by default for the scope and resource family. |

## Resolution

Agent Runs can pass `agent_runtime_profile_id` explicitly. If they do not,
MIOSA resolves defaults by scope and target family:

```text
explicit run profile
  -> project default
  -> workspace default
  -> tenant default
  -> no profile
```

The profile can be applied when creating or running sandboxes, computers, and
OpenComputer/BYOC agent sessions. Request-level `env` values override non-secret
profile env values for that single run.

## CLI

Agent Runtime Profiles have no dedicated `miosa` subcommand; drive them with
`miosa api` against the `/agent-runtime-profiles` endpoint.

```bash
miosa api GET /agent-runtime-profiles?workspace_id=ws_123

miosa api POST /agent-runtime-profiles -d '{
  "name": "lumen-claude",
  "runtime": "claude-code",
  "workspace_id": "ws_123",
  "project_id": "proj_123",
  "tools": ["filesystem", "shell", "browser", "artifacts"],
  "connectors": ["anthropic/lumen", "refero/managed"],
  "env": {"ANTHROPIC_MODEL": "claude-sonnet-4.6"},
  "policy": {"approval_required_for": ["deploy", "external_write"]},
  "is_default": true
}'

miosa api PUT /agent-runtime-profiles/arp_123 -d '{"runtime": "codex"}'
miosa api DELETE /agent-runtime-profiles/arp_123
```

## Next

- [Runtime Env](/docs/api-reference/runtime-env): Store inherited environment variables for tenant, workspace, project, and target scopes.
- [Agent Runs](/docs/api-reference/agent-runs): Dispatch prompts into a sandbox, computer, or custom runtime.
- [Agent orchestration](/docs/guides/agent-orchestration): Use runtime profiles with groups, events, artifacts, and multi-agent fanout.
