---
slug: "api-reference/sandboxes/create"
title: "Sandboxes API - create and run"
description: "Create a MIOSA sandbox, use the fused run endpoint, start a template app, get artifacts, and the custom-template boundary."
section: "API reference"
order: 12015
---

Base path: `/api/v1/sandboxes`

> **Note**
>
> All endpoints require `Authorization: Bearer &lt;api_key&gt;`.

## Create a Sandbox

**`POST /api/v1/sandboxes`**

Spawns a new sandbox VM. The API returns the sandbox record immediately after the create request is accepted; poll `GET /api/v1/sandboxes/{id}` or subscribe to events until `state` is `running` and `ready` is `true`.

The production alias `miosa-sandbox` resolves to a certified immutable artifact generation.

### Auth

Bearer token required.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `template_id` | string | No | Template to stage inside the sandbox image at create. Omit it for a sandbox with nothing staged. See templates below. |
| `size` | string | No | Canonical resource configuration. Defaults to `small` (`2 vCPU`, `4096 MiB`, `10240 MiB` disk). |
| `cpu_count` | integer | No | Exact vCPU compatibility input. Supply the full matching resource triple. |
| `memory_mb` | integer | No | Exact RAM override. It must match the selected canonical size. |
| `disk_size_mb` | integer | No | Exact disk override. It must match the selected canonical size. `disk_mb` is deprecated. |
| `timeout_sec` | integer | No | Active-session timeout. Defaults to `3600`; minimum `1`, maximum `86400`. |
| `always_on` | boolean | No | Disable timeout and idle-timeout enforcement until an explicit lifecycle action. Defaults to `false` and remains subject to policy. |
| `persistent` | boolean | No | Defaults to `true`. Preserve filesystem state across stop/timeout and allow resume. Set `false` only for disposable one-off jobs. |
| `idle_timeout_sec` | integer | No | Seconds without activity before the sandbox pauses. Left unset, it defaults to `0` (disabled), or `1800` (30 minutes) for accounts created on or after October 11, 2026; set `0` to disable. |
| `env` | object | No | Key-value env vars injected at boot. |
| `metadata` | object | No | Arbitrary caller-supplied metadata stored on the record. |
| `agent_runtime_profile_id` | UUID/string | No | Force a specific agent runtime profile for this sandbox. Alias: `agent_profile_id`. |
| `skip_agent_runtime_profile` | boolean | No | If `true`, do not apply the tenant/workspace default runtime profile. |
| `auto_start` | boolean | No | If `true`, MIOSA starts the selected template after the sandbox reaches `running`. Generated-app platforms usually keep this false, write files first, then call `/template/start`. |
| `workspace_id` | UUID | No | Existing MIOSA workspace that owns the sandbox. Defaults to the organization default workspace. |
| `workspace_slug` | string | No | Existing or auto-created workspace slug. |
| `workspace_name` | string | No | Workspace display name if auto-created. |
| `project_id` | UUID | No | Existing MIOSA project that owns the sandbox. Defaults to the workspace default project. |
| `project_slug` | string | No | Existing or auto-created project slug inside the workspace. |
| `project_name` | string | No | Project display name if auto-created. |
| `external_workspace_id` | string | No | Your customer/account/workspace ID. |
| `external_user_id` | string | No | Your end-user ID. |
| `external_project_id` | string | No | Your project/app/document ID. |

If a default agent runtime profile exists for the workspace or tenant, MIOSA
applies it automatically during create. Profile env vars are merged with request
env vars, request env wins on conflict, profile metadata is recorded under
`metadata.agent_runtime_profile`, and profile connectors are attached through
egress placeholder tokens rather than copied as plaintext secrets.

**Available templates:**

| `template_id` | Description |
|---------------|-------------|
| `miosa-sandbox` | The default sandbox, and the default when you omit `template_id`. Everything inside: Docker Engine 29.9 with `docker compose` and `docker buildx`, a live browser with CDP on a virtual display, the persistent code interpreter, Node, and Python. |

Every other built-in id is an alias of `miosa-sandbox`, kept so existing code keeps working; the [Sandbox Templates API](/docs/api-reference/sandbox-templates) lists them.
For a desktop, create a Computer with `template_type: "miosa-desktop"`; see [Computers](/docs/computers/overview).

### Request Headers

| Header | Description |
|--------|-------------|
| `Idempotency-Key` | Client-generated key (UUID recommended). Same key within 24 h returns the existing sandbox instead of creating a new one. |

### Response  -  `201 Created`
```json
{
  "id": "sbx_01j9xr2t4fk8me3n5q",
  "tenant_id": "tnt_abc123",
  "owner_id": "usr_def456",
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "project_id": "660e8400-e29b-41d4-a716-446655440001",
  "external_workspace_id": "acct_123",
  "external_user_id": "user-456",
  "external_project_id": "project_789",
  "template_id": "miosa-sandbox",
  "image_id": "miosa-sandbox",
  "state": "provisioning",
  "ready": false,
  "size": "small",
  "resource_contract": {
    "id": "sandbox/small@v1",
    "product": "sandbox",
    "size": "small",
    "version": "v1",
    "vcpus": 2,
    "memory_mb": 4096,
    "disk_size_mb": 10240
  },
  "cpu_count": 2,
  "memory_mb": 4096,
  "disk_size_mb": 10240,
  "preview_url": "https://sbx01j9x.sandbox.miosa.ai",
  "timeout_sec": 3600,
  "timeout_remaining_ms": null,
  "always_on": false,
  "persistent": true,
  "idle_timeout_sec": 0,
  "total_runtime_sec": null,
  "metadata": {},
  "created_at": "2026-04-25T10:00:00Z",
  "started_at": null,
  "ready_at": null,
  "destroyed_at": null
}
```

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `INVALID_TEMPLATE` | `template_id` is not a recognized template. |
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits to provision the VM. |
| 429 | `SANDBOX_LIMIT_REACHED` | Tenant has reached the concurrent sandbox limit for its plan (Developer 10, Business 250, Enterprise 500). |
| 422 | `VALIDATION_ERROR` | Invalid field values (for example, a resource request above the tenant's plan cap). |

---

## Create and Run

**`POST /api/v1/sandboxes/run`**

Creates a sandbox, waits for it to become `running`, executes the first command, and returns the sandbox record plus the command result in one request. Use this for agent loops where the first useful action is "create a sandbox and run code now."

This is the production fast path for agents that need a sandbox and an immediate first command.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/run \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: run-2026-06-09-001" \
  -d '{
    "template_id": "miosa-sandbox",
    "command": "python3 -c \"print(1+1)\"",
    "size": "small",
    "timeout_sec": 300,
    "wait_timeout_ms": 30000,
    "metadata": { "agent_run": "abc123" }
  }'
```

### Request Body

`/run` accepts the same sandbox creation fields as `POST /api/v1/sandboxes`, plus:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `command` | string | Yes | First command to execute after the sandbox reaches `running`. |
| `cwd` | string | No | Working directory for the command. Defaults to `/workspace`. |
| `timeout` | integer | No | Command timeout in milliseconds. |
| `wait_timeout_ms` | integer | No | Max time to wait for sandbox readiness before returning `504 SANDBOX_READY_TIMEOUT`. |

### Response  -  `201 Created`

```json
{
  "data": {
    "id": "sbx_01j9xr2t4fk8me3n5q",
    "state": "running",
    "ready": true,
    "template_id": "miosa-sandbox",
    "cpu_count": 2,
    "memory_mb": 4096
  },
  "exec": {
    "stdout": "2\n",
    "stderr": "",
    "exit_code": 0
  },
  "timings": {
    "server_wait_and_exec_ms": 512
  }
}
```

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `MISSING_PARAM` | `command` is missing. |
| 400 | `INVALID_TEMPLATE` | `template_id` is not recognized. |
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits to provision the VM. |
| 429 | `SANDBOX_LIMIT_REACHED` | Tenant has reached the concurrent sandbox limit. |
| 409 | `SANDBOX_NOT_RUNNING` | The sandbox failed to reach a running state for exec. |
| 502 | `SANDBOX_BOOT_FAILED` | VM boot failed before the command could run. |
| 502 | `AGENT_UNAVAILABLE` | The sandbox agent was not reachable for exec. |
| 504 | `SANDBOX_READY_TIMEOUT` | The sandbox did not become ready before `wait_timeout_ms`. |

---

## Start a Template App

**`POST /api/v1/sandboxes/{id}/template/start`**

A sandbox created from a template that has a start command does not wait on the app at create: nothing has started it yet, so a `wait: true` create returns as soon as the sandbox is running instead of timing out.
The template's own readiness probe applies when you call `/template/start`; a probe you pass to create always wins.

After your platform writes generated files into `/workspace`, call this endpoint to run the selected template lifecycle. MIOSA runs the template install command, launches the start command in the background, stores PID/log paths, and returns the preview URL.
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/template/start \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"install":true}'
```

Response:
```json
{
  "data": {
    "status": "started",
    "template_id": "nextjs",
    "workdir": "/workspace",
    "preview_port": 3000,
    "preview_url": "https://abc12345.sandbox.miosa.ai",
    "logs_path": "/tmp/miosa-run/template.log",
    "pid_path": "/tmp/miosa-run/template.pid",
    "artifact_paths": ["/workspace"]
  }
}
```

You can override `install_command`, `start_command`, `port`, or `workdir` in the request body when your generated project needs a custom command.

---

## Get Artifacts

**`GET /api/v1/sandboxes/{id}/artifacts`**

Returns the template artifact contract and current lifecycle metadata.
```bash
curl https://api.miosa.ai/api/v1/sandboxes/{id}/artifacts \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

The response includes `preview.url`, `preview.port`, artifact paths, and template lifecycle log/PID paths.

A create request with white-label attribution and env:

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template_id": "miosa-sandbox",
    "workspace_slug": "acme-corp",
    "workspace_name": "Acme Corp",
    "project_slug": "lead-magnet",
    "project_name": "Lead Magnet",
    "external_workspace_id": "acct_123",
    "external_user_id": "user-456",
    "external_project_id": "project_789",
    "size": "small",
    "env": {"MY_VAR": "hello"}
  }'
```

---

## Custom templates

The canonical public V1 contract exposes only promoted templates returned by `GET /api/v1/templates`.
Standalone custom-template build, validation, log, retry, and cancellation endpoints are not part of that contract.
Do not make a public integration depend on those operations.
