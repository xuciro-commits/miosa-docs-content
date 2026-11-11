---
slug: "api-reference/computers"
title: "Computers API"
description: "API reference for creating, managing, and controlling MIOSA computers - full lifecycle, desktop control, files, metrics, and env vars."
section: "API reference"
order: 12029
---

Computers are persistent desktop VMs with a Linux desktop, streamed desktop access, and APIs for exec, files, apps, metrics, and desktop control. Base path: `/api/v1/computers`.

Verbs supported: **GET** (list/get/config), **POST** (create/start/stop/restart/resize), **PATCH** (update), **PUT** (update), **DELETE** (delete).

> **Caution**
>
> Selected mutation endpoints (sandbox templates and migration) accept an `Idempotency-Key` header. Use a UUID v4. Repeating the same key within 24 hours returns the original response without re-executing the mutation.

> **Caution**
>
> Rate limits vary by key, tenant policy, and endpoint family. Exceeding the limit returns HTTP 429 with a `Retry-After` header; honor it when you back off.

---

## Create a Computer

**`POST /api/v1/computers`**

Creates a new computer and begins provisioning.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Display name |
| `template_type` | string | Yes | Template (`"miosa-desktop"`) |
| `size` | string | No | `"small"` (4GB/1CPU), `"medium"` (8GB/2CPU), `"large"` (16GB/4CPU). Default: `"small"` |
| `selected_apps` | string[] | No | Applications to install after boot |
| `workspace_id` | UUID | No | Workspace to assign this computer to. Defaults to the tenant's default workspace. |
| `workspace_slug` | string | No | Existing or auto-created workspace slug. |
| `workspace_name` | string | No | Workspace display name if auto-created. |
| `project_id` | UUID | No | Project to assign this computer to. Defaults to the workspace default project. |
| `project_slug` | string | No | Existing or auto-created project slug. |
| `project_name` | string | No | Project display name if auto-created. |
| `external_workspace_id` | string | No | Your own customer/account/workspace ID. |
| `external_user_id` | string | No | Your own end-user ID. |
| `external_project_id` | string | No | Your own project/app/document ID. |
| `agent_runtime_profile_id` | UUID/string | No | Force a specific agent runtime profile for this computer. Alias: `agent_profile_id`. |
| `skip_agent_runtime_profile` | boolean | No | If `true`, do not apply the tenant/workspace default runtime profile. |

If a default agent runtime profile exists for the workspace or tenant and it
applies to computers, MIOSA applies it automatically. Profile env vars are
stored in the encrypted computer env store and synced into the VM after OSA is
healthy. Profile connectors are bound through MIOSA egress as opaque
`miosa-tok-*` placeholder tokens.

### Response  -  `201 Created`
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "tenant_id": "...",
  "owner_user_id": "...",
  "name": "my-computer",
  "slug": "my-computer",
  "template_type": "miosa-desktop",
  "size": "small",
  "vcpus": 1,
  "memory_mb": 4096,
  "status": "provisioning",
  "vm_id": null,
  "workspace_id": "660e8400-e29b-41d4-a716-446655440001",
  "project_id": "770e8400-e29b-41d4-a716-446655440002",
  "external_workspace_id": null,
  "external_user_id": null,
  "external_project_id": null,
  "sandbox_url": "https://my-computer.sandbox.miosa.ai",
  "desktop_url": "https://my-computer.computer.miosa.ai/desktop/index.html",
  "selected_apps": [],
  "settings": {},
  "ai_config": {},
  "agent_session_id": null,
  "agent_status": null,
  "resolution": "1280x720",
  "auto_stop": 0,
  "created_at": "2026-04-11T00:00:00Z",
  "updated_at": "2026-04-11T00:00:00Z"
}
```

`sandbox_url` is a legacy preview field on the Computer response. Use `desktop_url` for the desktop stream and the `/computers/{id}/urls` endpoint for short-lived desktop and terminal URLs.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `name is required` | Missing name or template_type |
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits to create |
| 404 | `AGENT_RUNTIME_PROFILE_NOT_FOUND` | Explicit profile ID does not exist in this tenant |
| 422 | `TENANT_RESOLUTION_FAILED` | Cannot determine tenant for user |
| 422 | `AGENT_RUNTIME_PROFILE_NOT_APPLICABLE` | Explicit profile does not apply to computers |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-computer",
    "template_type": "miosa-desktop",
    "size": "small",
    "workspace_slug": "acme-corp",
    "workspace_name": "Acme Corp",
    "project_slug": "records-portal",
    "project_name": "Records Portal",
    "external_workspace_id": "acct_123",
    "external_user_id": "user-456",
    "external_project_id": "project_789",
    "agent_runtime_profile_id": "9f4b3d8e-0000-0000-0000-000000000000"
  }'
```

---

## List Computers

**`GET /api/v1/computers`**

Returns all computers belonging to the authenticated user's tenant.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `workspace_id` | UUID | Filter computers to a specific workspace. Omit to return computers across all workspaces. |
| `project_id` | UUID | Filter computers to a specific project. |
| `external_workspace_id` | string | Filter by your customer/account/workspace ID. |
| `external_user_id` | string | Filter by your end-user ID. |
| `external_project_id` | string | Filter by your project/app/document ID. |

### Response  -  `200 OK`
```json
{
  "computers": [
    {
      "id": "...",
      "name": "my-computer",
      "status": "running",
      "size": "small",
      "created_at": "2026-04-11T00:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
# All computers
curl https://api.miosa.ai/api/v1/computers \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Filtered to a workspace
curl "https://api.miosa.ai/api/v1/computers?workspace_id=660e8400-e29b-41d4-a716-446655440001" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Filtered to a project
curl "https://api.miosa.ai/api/v1/computers?project_id=770e8400-e29b-41d4-a716-446655440002" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get a Computer

**`GET /api/v1/computers/{id}`**

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | UUID | Computer ID |

### Response  -  `200 OK`

Full computer object (same as create response, with current status).

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `invalid computer id` | Not a valid UUID |
| 404 | `computer not found` | Does not exist or belongs to different tenant |
```bash
curl https://api.miosa.ai/api/v1/computers/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Update a Computer

**`PATCH /api/v1/computers/{id}`**

Currently supports updating `agent_session_id` only.

### Request Body

| Field | Type | Description |
|-------|------|-------------|
| `agent_session_id` | string | Link an AI agent session |

### Response  -  `200 OK`

Updated computer object.
```bash
curl -X PATCH https://api.miosa.ai/api/v1/computers/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"agent_session_id": "session-uuid"}'
```

---

## Delete a Computer

**`DELETE /api/v1/computers/{id}`**

Permanently destroys the VM and removes the computer record.

### Response  -  `202 Accepted`

Returns the computer object; its `state` reflects the destroy in progress.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 403 | `not a member of this computer` | No access to this computer |
| 404 | `computer not found` | Does not exist |
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Start a Computer

**`POST /api/v1/computers/{id}/start`**

Resumes a stopped or paused computer.

### Response  -  `200 OK`

Updated computer object with `status: "running"`.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits |
| 409 | `computer cannot be started from its current status` | Not in a startable state |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/start \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Stop a Computer

**`POST /api/v1/computers/{id}/stop`**

Pauses a running computer. Can be restarted later.

### Response  -  `200 OK`

Updated computer object with `status: "stopped"`.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 409 | `computer cannot be stopped from its current status` | Not running |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/stop \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Restart a Computer

**`POST /api/v1/computers/{id}/restart`**

Stops and immediately restarts a running computer.

### Response  -  `200 OK`

Updated computer object.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 409 | `computer must be running to restart` | Not running |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/restart \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get Auto-Stop Configuration

**`GET /api/v1/computers/{id}/auto-stop`**

### Response  -  `200 OK`
```json
{
  "auto_stop_seconds": 3600,
  "enabled": true
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/auto-stop \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Update Auto-Stop Configuration

**`PATCH /api/v1/computers/{id}/auto-stop`**

### Request Body

| Field | Type | Description |
|-------|------|-------------|
| `seconds` | integer | Seconds of inactivity before auto-stop. `0` to disable. |

### Response  -  `200 OK`

Updated auto-stop configuration.
```bash
curl -X PATCH https://api.miosa.ai/api/v1/computers/{id}/auto-stop \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"seconds": 3600}'
```

---

## Get Desktop Stream Credentials

**`GET /api/v1/computers/{id}/vnc-credentials`**

Returns credentials for direct desktop streaming access.

### Response  -  `200 OK`
```json
{
  "desktop_url": "https://my-computer.computer.miosa.ai/desktop/index.html",
  "ws_url": "wss://my-computer.computer.miosa.ai/ws/vnc/550e8400-e29b-41d4-a716-446655440000?auth=stream_token",
  "token": "stream_auth_token",
  "expires_at": 1712700060,
  "computer_id": "550e8400-e29b-41d4-a716-446655440000",
  "slug": "my-computer"
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/vnc-credentials \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get Stream Token

**`POST /api/v1/computers/{id}/stream-token`**

Returns a time-limited auth token for desktop and terminal WebSocket connections.

### Response  -  `200 OK`
```json
{
  "token": "stream_auth_token",
  "expires_at": 1712700060
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/stream-token \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get Computer URLs

**`GET /api/v1/computers/{id}/urls`**

Returns all connection URLs for the computer.

### Response  -  `200 OK`
```json
{
  "desktop_url": "https://my-computer.computer.miosa.ai/desktop/index.html",
  "stream_url": "https://my-computer.computer.miosa.ai/desktop/index.html",
  "terminal_url": "wss://my-computer.computer.miosa.ai/ws/terminal/550e8400-e29b-41d4-a716-446655440000",
  "computer_id": "550e8400-e29b-41d4-a716-446655440000"
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/urls \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## List Installed Apps

**`GET /api/v1/computers/{id}/apps`**

Returns the list of applications installed on the computer.

### Response  -  `200 OK`

Response containing the list of installed applications.
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/apps \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Resize a Computer

**`POST /api/v1/computers/{id}/resize`**

Changes the vCPU, memory, and disk allocation. A running computer is resized in place.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cpu` | integer | No | Target vCPU count |
| `memory_mb` | integer | No | Target memory in MiB |
| `disk_gb` | integer | No | Target disk in GiB |

### Response  -  `200 OK`

Updated computer object with the new allocation.

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/resize \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"cpu": 2, "memory_mb": 8192, "disk_gb": 32}'
```

---

## Move a Computer

**`POST /api/v1/computers/{id}/move`**

Moves a computer to a different workspace in the same organization.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `workspace_id` | string | Yes | Target workspace ID |

### Response  -  `200 OK`

Updated computer object with the new `workspace_id`.

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/move \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workspace_id": "550e8400-e29b-41d4-a716-446655440001"}'
```

---

## Get Metrics

**`GET /api/v1/computers/{id}/metrics`**

Returns time-series CPU, RAM, and credit-burn metrics. Useful for dashboards and auto-stop logic.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `window` | string | Time window: `"1h"` (default), `"24h"`, `"7d"` |

### Response  -  `200 OK`

```json
{
  "cpu_percent": [
    {"ts": "2026-05-17T10:00:00Z", "value": 12.4},
    {"ts": "2026-05-17T10:01:00Z", "value": 8.1}
  ],
  "memory_percent": [
    {"ts": "2026-05-17T10:00:00Z", "value": 45.2}
  ],
  "credits_per_hour": 3
}
```

```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/metrics?window=1h" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Env Vars

Encrypted per-computer environment variables. Values are decrypted at boot and injected into the VM environment.

### List

**`GET /api/v1/computers/{id}/env`**

```bash
curl https://api.miosa.ai/api/v1/computers/{id}/env \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

### Create

**`POST /api/v1/computers/{id}/env`**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Variable name (e.g. `DATABASE_URL`) |
| `value` | string | Yes | Plaintext value  -  encrypted at rest |

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/env \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "DATABASE_URL", "value": "postgres://user:pass@host/db"}'
```

### Update

**`PATCH /api/v1/computers/{id}/env/{name}`**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | string | Yes | New plaintext value |

### Delete

**`DELETE /api/v1/computers/{id}/env/{name}`**

Returns `200 OK`.

---

## Port Exposure

Control per-port visibility for services running inside the computer.

### List Ports

**`GET /api/v1/computers/{id}/ports`**

### Expose a Port

**`POST /api/v1/computers/{id}/ports`**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `port` | integer | Yes | Port number (1-65535) |
| `visibility` | string | Yes | `"public"`, `"private"`, or `"protected"` |

### Update Port Visibility

**`PATCH /api/v1/computers/{id}/ports/{port}`**

### Remove Port

**`DELETE /api/v1/computers/{id}/ports/{port}`**

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/ports \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"port": 3000, "visibility": "public"}'
```

---

## Common Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `invalid computer id` | Path parameter is not a valid UUID |
| 402 | `INSUFFICIENT_CREDITS` | Not enough credits to start or create |
| 403 | `not a member of this computer` | No access to this computer |
| 404 | `computer not found` | Does not exist or belongs to a different tenant |
| 409 | `COMPUTER_NOT_RUNNING` | Operation requires a running computer |
| 502 | `AGENT_UNAVAILABLE` | Computer command service unavailable |

---

## See also

- [Desktop API](/docs/api-reference/desktop)  -  screenshot, click, type, scroll, drag, windows
- [Files API](/docs/api-reference/files)  -  read, write, stat, mkdir, rename, copy, chmod, download
- [Exec API](/docs/api-reference/exec)  -  bash and Python execution
- [Snapshots API](/docs/api-reference/snapshots)  -  create, restore, and delete checkpoints
- [Events (SSE)](/docs/api-reference/events)  -  computer lifecycle event stream
- [Regions](/docs/api-reference/regions)  -  available regions for computer placement
