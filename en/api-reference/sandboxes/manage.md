---
slug: "api-reference/sandboxes/manage"
title: "Sandboxes API - manage"
description: "List, get, destroy, pause, resume, restart, fork, extend the timeout, and read usage for MIOSA sandboxes."
section: "API reference"
order: 12016
---

Base path: `/api/v1/sandboxes`

> **Note**
>
> All endpoints require `Authorization: Bearer &lt;api_key&gt;`.

## List Sandboxes

**`GET /api/v1/sandboxes`**

Returns all sandboxes belonging to the authenticated tenant.

### Query Parameters

| Parameter | Type | Description |
|---|---|---|
| `workspace_id` | UUID | Filter to one MIOSA workspace |
| `project_id` | UUID | Filter to one MIOSA project |
| `external_workspace_id` | string | Filter by your customer/account ID |
| `external_user_id` | string | Filter by your end-user ID |
| `external_project_id` | string | Filter by your project/app/document ID |
| `state` | string | Filter by lifecycle state: `provisioning`, `running`, `paused`, `destroyed`, `error`. |

### Auth

Bearer token required.

### Response  -  `200 OK`
```json
{
  "data": [
    {
      "id": "sbx_01j9xr2t4fk8me3n5q",
      "template_id": "miosa-sandbox",
      "state": "running",
      "cpu_count": 2,
      "memory_mb": 4096,
      "created_at": "2026-04-25T10:00:00Z"
    }
  ]
}
```
```bash
curl "https://api.miosa.ai/api/v1/sandboxes?state=running" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get a Sandbox

**`GET /api/v1/sandboxes/{id}`**

### Auth

Bearer token required.

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Sandbox ID. |

### Response  -  `200 OK`

Full sandbox object (same shape as the create response with current `state`).

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 | `NOT_FOUND` | Sandbox does not exist or belongs to a different tenant. |
```bash
curl https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Destroy a Sandbox

**`DELETE /api/v1/sandboxes/{id}`**

Permanently deletes the sandbox, removes saved state, and settles billing. Use `POST /pause` when the user is done for now but may resume later.

### Auth

Bearer token required.

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Sandbox ID. |

### Response  -  `200 OK`
```json
{
  "id": "sbx_01j9xr2t4fk8me3n5q",
  "state": "destroyed",
  "total_runtime_sec": 42
}
```

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 | `NOT_FOUND` | Sandbox does not exist or belongs to a different tenant. |
```bash
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Pause and Resume

**`POST /api/v1/sandboxes/{id}/pause`** suspends guest vCPU execution for a running persistent sandbox while preserving its workspace.
It returns `409` if the sandbox is not running.

**`POST /api/v1/sandboxes/{id}/resume`** transitions a paused sandbox back to `running` and returns `409` if it is not paused.
Command and file operations can automatically resume a paused persistent sandbox before dispatch.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/pause \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/resume \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Idempotency-Key: resume-session-001"
```

---

## Restart

**`POST /api/v1/sandboxes/{id}/restart`** power-cycles a running sandbox under the same id.
It stops the sandbox keeping its disk, then boots that disk again.
Files, packages and enabled services survive; running processes do not.
Use it when a sandbox stops answering.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/restart \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

| Status | Code | Meaning |
|---|---|---|
| `200` | | Restarted: `{"id", "state": "running", "restarted": true}`. |
| `202` | | Still booting: `state` is `resuming` or `starting`; poll `GET /sandboxes/{id}`. |
| `409` | `SANDBOX_NOT_RUNNING` | Only a running sandbox restarts; use `POST /resume`. |
| `409` | `RESTART_NOT_POWER_CYCLED` | Its host stopped it with a memory checkpoint, so it was resumed from memory: running again, processes not restarted. |
| `502` | `RESTART_BOOT_FAILED` | It stopped and kept its disk but did not boot; `POST /resume` boots it again. |
| `503` | `RESTART_STOP_IN_PROGRESS` | It is still stopping; resume it once it is `stopped`. |

---

## Fork a Sandbox

**`POST /api/v1/sandboxes/{id}/fork`** creates a new sandbox from a copy-on-write snapshot of a running sandbox.
The optional body accepts `timeout_sec` and `template_id` for the new sandbox.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/fork \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: fork-experiment-001" \
  -d '{"timeout_sec":3600}'
```

Canonical public V1 exposes standalone snapshot create, list, inspect, delete, and restore operations.
Use fork when a customer needs an independent branch of current running state without managing the intermediate snapshot directly.

---

## Extend the Timeout

**`POST /api/v1/sandboxes/{id}/extend`**

Replaces the active-session timeout with `timeout_sec` from 1 through 86400.
The operation does not add duration to the old value.
Omitting the body preserves the current timeout.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/extend \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"timeout_sec":7200}'
```

---

## Get Sandbox Usage

**`GET /api/v1/sandboxes/{id}/usage`**

Returns measured runtime, provisioned vCPU time, active CPU, network traffic, estimated cost, and timeout visibility.

```json
{
  "data": {
    "sandbox_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "state": "running",
    "runtime_sec": 47,
    "provisioned_vcpu_ms": 94000,
    "active_cpu_ms": 12000,
    "network_ingress_bytes": 2048,
    "network_egress_bytes": 4096,
    "measurement_status": {
      "active_cpu": "measured",
      "network": "measured",
      "provisioned_resources": "measured"
    },
    "estimated_cost_cents": 1,
    "timeout_sec": 3600,
    "timeout_remaining_ms": 3553000
  }
}
```

Interpret nullable measurements through `measurement_status`.
`unavailable` means no trustworthy value exists and the related field is `null`, not zero.
`stale` means the retained value is not current and must be labeled as such.
`timeout_remaining_ms` can be `null` before start or when timeout enforcement is disabled.
