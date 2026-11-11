---
slug: "api-reference/workspaces"
title: "Workspaces API"
description: "API reference for creating and managing MIOSA workspaces - the customer/client layer inside an organization."
section: "Organization & billing"
order: 13000
---

Workspaces group resources inside an organization. In a white-label platform, a workspace usually represents one downstream customer or client, such as `Acme Corp` inside the `Lumen` organization. Projects live inside workspaces, and sandboxes, computers, deployments, databases, storage buckets, volumes, functions, jobs, and domains belong to projects.

Base path: `/api/v1/workspaces`

> **Note**
>
> A **default** workspace is created automatically when you sign up. Resources are assigned to the default workspace and its default project unless you specify workspace/project ownership at creation time.

---

## Quick Start
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa(); // reads MIOSA_API_KEY

// Create a workspace for a downstream customer
const ws = await client.workspaces.create({
  name: 'Acme Corp',
  slug: 'acme-corp',
  externalWorkspaceId: 'acct_123',
});

// Create a computer inside a project in that workspace
const computer = await client.computers.create({
  name: 'workspace-builder',
  template_type: 'miosa-desktop',
  workspaceId: ws.id,
  projectSlug: 'lead-magnet',
  projectName: 'Lead Magnet',
});

// List computers in the workspace
const { data } = await client.workspaces.listComputers(ws.id);
```
```bash
# curl equivalent
curl -X POST https://api.miosa.ai/api/v1/workspaces \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Acme Corp", "slug": "acme-corp", "external_workspace_id": "acct_123"}'
```

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/workspaces` | List workspaces for the tenant |
| `POST` | `/workspaces` | Create a workspace |
| `GET` | `/workspaces/{id}` | Get a workspace |
| `PATCH` | `/workspaces/{id}` | Update a workspace |
| `PUT` | `/workspaces/{id}/settings` | Update workspace-level resource limits |
| `DELETE` | `/workspaces/{id}` | Delete a workspace |
| `GET` | `/workspaces/{id}/computers` | List computers in a workspace |
| `GET` | `/workspaces/{id}/stats` | Computer counts and resource totals |
| `GET` | `/workspaces/{id}/usage` | Time-series credit and compute usage |
| `GET` | `/workspaces/{id}/projects` | List projects in a workspace |

---

## List Workspaces

**`GET /api/v1/workspaces`**

Returns all workspaces belonging to the authenticated tenant.

### Response  -  `200 OK`
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "tenant_id": "...",
      "name": "default",
      "slug": "default",
      "external_workspace_id": null,
      "description": null,
      "metadata": {},
      "is_default": true,
      "computer_count": 3,
      "created_at": "2026-04-11T00:00:00Z",
      "updated_at": "2026-04-11T00:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
curl https://api.miosa.ai/api/v1/workspaces \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Create a Workspace

**`POST /api/v1/workspaces`**

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Human-readable name (1-120 chars) |
| `slug` | string | No | URL-safe identifier. Auto-derived from `name` if omitted. Pattern: `[a-z0-9][a-z0-9-]{0,79}` |
| `external_workspace_id` | string | No | Your customer/account/workspace ID. Globally unique inside this MIOSA organization. |
| `description` | string | No | Optional description |
| `metadata` | object | No | Caller metadata stored on the workspace. |

### Response  -  `201 Created`

Full workspace object (same shape as list items above).

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 422 | `has already been taken` | Slug already exists in tenant |
| 422 | `must start with alphanumeric and contain only a-z 0-9 -` | Invalid slug format |
```bash
curl -X POST https://api.miosa.ai/api/v1/workspaces \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme Corp",
    "slug": "acme-corp",
    "external_workspace_id": "acct_123",
    "description": "Client workspace for Acme Corp"
  }'
```

---

## Get a Workspace

**`GET /api/v1/workspaces/{id}`**

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | UUID | Workspace ID |

### Response  -  `200 OK`

Full workspace object.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 403 | `forbidden` | Workspace belongs to a different tenant |
| 404 | `workspace not found` | Does not exist |
```bash
curl https://api.miosa.ai/api/v1/workspaces/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Update a Workspace

**`PATCH /api/v1/workspaces/{id}`**

Update mutable workspace fields. Prefer treating the slug as stable once resources and URLs exist.

### Request Body

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | New display name |
| `slug` | string | New URL-safe slug. Must remain unique inside the organization. |
| `description` | string | New description (`null` to clear) |
| `external_workspace_id` | string | Your customer/account/workspace ID |
| `metadata` | object | Replacement metadata map |

### Response  -  `200 OK`

Updated workspace object.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 403 | `forbidden` | Wrong tenant |
| 404 | `workspace not found` | Does not exist |
```bash
curl -X PATCH https://api.miosa.ai/api/v1/workspaces/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "production-us-east"}'
```

---

## Delete a Workspace

**`DELETE /api/v1/workspaces/{id}`**

Deletes an empty workspace. Fails if the workspace contains computers or projects.

### Response  -  `200 OK`
```json
{ "deleted": true }
```

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 403 | `forbidden` | Wrong tenant |
| 404 | `workspace not found` | Does not exist |
| 409 | `workspace has computers` | Remove or move resources first |
```bash
curl -X DELETE https://api.miosa.ai/api/v1/workspaces/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Update Workspace Settings

**`PUT /api/v1/workspaces/{id}/settings`**

Applies resource limits and default preferences for the workspace. These limits cap what any computer inside the workspace can do. Tenant-level limits still apply as a ceiling.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `max_computers` | integer | No | Maximum number of computers allowed in this workspace. `null` = no workspace-level cap (tenant limit applies). |
| `max_computer_size` | string | No | Maximum `size` value a computer in this workspace may be created with: `"small"`, `"medium"`, or `"large"`. |
| `default_computer_size` | string | No | Default `size` applied when creating a computer without an explicit `size`. |
| `default_auto_stop_seconds` | integer | No | Auto-stop idle timeout (seconds) applied to new computers by default. `0` = no auto-stop. |
| `allowed_template_types` | string[] | No | Allowlist of template names that can be used in this workspace. Empty array = all templates allowed. |
| `monthly_spend_cap_cents` | integer | No | Monthly cap, in credits (1 credit = 1 US cent), on hosted sandbox spend in this workspace. `null` clears it. |
| `monthly_total_spend_cap_cents` | integer | No | Monthly cap, in credits, on everything settled against this workspace: sandboxes, computers, deployments, databases, storage, snapshots, backups and functions. `null` clears it. |
| `spend_cap_action` | string | No | What reaching a cap does: `"pause"` (the default) refuses new resources in the workspace with HTTP 402 and stops what is running there, `"alert"` only sends threshold alerts. |
| `entity_spend_caps` | object | No | Monthly caps per `entity` tag, for example `{"customer-a": 5000}`. |

Nothing is destroyed and nothing resumes by itself. The setting is named `spend_cap_action`, not `action`.

### Response  -  `200 OK`
```json
{
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "settings": {
    "max_computers": 10,
    "max_computer_size": "large",
    "default_computer_size": "small",
    "default_auto_stop_seconds": 3600,
    "allowed_template_types": []
  },
  "updated_at": "2026-05-17T10:00:00Z"
}
```

---

## Domains

Workspaces do not own a preview domain. Preview domains are an organization
setting, and per-deployment custom domains are attached to the deployment:

- [Custom Domains API](/docs/api-reference/custom-domains) - the organization preview domain and deployment/computer custom domains.

---

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 403 | `FORBIDDEN` | The caller is not an organization owner or admin (`Only workspace admins may update workspace limits`) |
| 403 | `forbidden` | Wrong tenant |
| 404 | `workspace not found` | Does not exist |
| 422 | `VALIDATION_FAILED` | Unknown size value or invalid limit |
| 422 | `UNKNOWN_WORKSPACE_SETTING` | A setting name the API does not know. The `message` names the closest accepted key (sending `action` answers `Did you mean "spend_cap_action"?`) and `allowed` lists every accepted setting. |
```bash
curl -X PUT https://api.miosa.ai/api/v1/workspaces/{id}/settings \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "max_computers": 10,
    "max_computer_size": "large",
    "default_auto_stop_seconds": 3600
  }'
```

---

## List Computers in a Workspace

**`GET /api/v1/workspaces/{id}/computers`**

### Response  -  `200 OK`
```json
{
  "data": [
    {
      "id": "...",
      "name": "runner-1",
      "status": "running",
      "size": "small",
      "template_type": "miosa-desktop",
      "created_at": "2026-04-11T00:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
curl https://api.miosa.ai/api/v1/workspaces/{id}/computers \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## List Projects in a Workspace

**`GET /api/v1/workspaces/{id}/projects`**

Returns the projects owned by a workspace. Use this for client dashboards before listing the sandboxes, computers, deployments, and databases inside a project.

### Response - `200 OK`
```json
{
  "data": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "tenant_id": "...",
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "external_workspace_id": "acct_123",
      "external_project_id": "project_789",
      "name": "Lead Magnet",
      "slug": "lead-magnet",
      "description": null,
      "metadata": {},
      "created_at": "2026-05-18T10:00:00Z",
      "updated_at": "2026-05-18T10:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
curl https://api.miosa.ai/api/v1/workspaces/{id}/projects \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get Workspace Stats

**`GET /api/v1/workspaces/{id}/stats`**

Returns a snapshot of computer counts grouped by status and the aggregate resource footprint for the workspace.

### Response  -  `200 OK`
```json
{
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "computers": {
    "total": 5,
    "by_status": {
      "running": 3,
      "stopped": 2,
      "provisioning": 0,
      "error": 0
    }
  },
  "resources": {
    "total_vcpus": 7,
    "total_memory_mb": 28672,
    "total_disk_gb": 160
  },
  "settings": {
    "max_computers": 10,
    "max_computer_size": "large",
    "default_auto_stop_seconds": 3600
  }
}
```
```bash
curl https://api.miosa.ai/api/v1/workspaces/{id}/stats \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get Workspace Usage

**`GET /api/v1/workspaces/{id}/usage`**

Returns time-series credit consumption and compute-hour totals for the workspace over a given window. Use this for per-workspace cost attribution dashboards.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `window` | string | `"24h"` (default), `"7d"`, `"30d"` |
| `granularity` | string | `"hour"` (default), `"day"`  -  bucket size for the time-series data |

### Response  -  `200 OK`
```json
{
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "window": "7d",
  "granularity": "day",
  "total_credits_used": 1284,
  "total_compute_hours": 42.5,
  "series": [
    {
      "ts": "2026-05-10T00:00:00Z",
      "credits_used": 198,
      "compute_hours": 6.6
    },
    {
      "ts": "2026-05-11T00:00:00Z",
      "credits_used": 211,
      "compute_hours": 7.0
    }
  ]
}
```

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 403 | `forbidden` | Wrong tenant |
| 404 | `workspace not found` | Does not exist |
| 422 | `VALIDATION_FAILED` | Unknown `window` or `granularity` value |
```bash
curl "https://api.miosa.ai/api/v1/workspaces/{id}/usage?window=7d&granularity=day" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Common Recipes

### Move all computers to a new workspace

Computers do not yet support a `move` operation on workspace assignment via `PATCH /computers/{id}`. Use the `POST /computers/{id}/move` endpoint instead:
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{computer_id}/move \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workspace_id": "target-workspace-id"}'
```

### Enumerate all workspaces and their sizes
```typescript
const { data: workspaces } = await client.workspaces.list();
for (const ws of workspaces) {
  console.log(`${ws.name}: ${ws.computerCount} computers`);
}
```

### Default workspace ID lookup

If you need the default workspace ID programmatically:
```typescript
const { data } = await client.workspaces.list();
const defaultWs = data.find(ws => ws.isDefault);
console.log(defaultWs.id);
```

---

## Common Errors

| Status | Code | Cause |
|--------|------|-------|
| 403 | `forbidden` | Workspace belongs to a different tenant |
| 404 | `workspace not found` | Does not exist |
| 409 | `workspace has computers` | Cannot delete a workspace that contains computers |
| 422 | `VALIDATION_FAILED` | Name or slug failed schema validation |

---

## See also

- [Computers API](/docs/api-reference/computers)  -  computers are scoped to workspaces
- [Projects API](/docs/api-reference/projects)  -  projects live inside workspaces
- [Error Codes](/docs/api-reference/errors)  -  `VALIDATION_FAILED` and general errors
