---
slug: "api-reference/projects"
title: "Projects API"
description: "API reference for creating and managing projects inside MIOSA workspaces."
section: "API reference"
order: 12044
---

A **Project** is the app, website, lead magnet, document, workflow, or customer build inside a workspace. Projects own sandboxes, computers, deployments, databases, storage buckets, volumes, functions, jobs, auth configuration, integrations, and custom domains.

Base path: `/api/v1/projects`

> **Note**
>
> If a resource create call includes `project_slug`, `project_name`, or `external_project_id`, MIOSA resolves or creates the project before creating the resource. You can also create projects explicitly with this API.

## Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/projects` | List projects for the organization |
| `POST` | `/projects` | Create a project inside a workspace |
| `GET` | `/projects/{id}` | Get one project |
| `PATCH` | `/projects/{id}` | Update a project |
| `GET` | `/workspaces/{id}/projects` | List projects in one workspace |

---

## List Projects

**`GET /api/v1/projects`**

### Query Parameters

| Parameter | Type | Description |
|---|---|---|
| `workspace_id` | UUID | Limit results to one workspace |

### Response - `200 OK`
```json
{
  "data": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "tenant_id": "tnt_abc123",
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "external_workspace_id": "acct_123",
      "external_project_id": "project_789",
      "name": "Lead Magnet",
      "slug": "lead-magnet",
      "description": null,
      "metadata": {},
      "settings": {},
      "created_at": "2026-05-18T10:00:00Z",
      "updated_at": "2026-05-18T10:00:00Z"
    }
  ],
  "total": 1
}
```

```bash
curl "https://api.miosa.ai/api/v1/projects?workspace_id=550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Create a Project

**`POST /api/v1/projects`**

A project must belong to a workspace. Supply one of `workspace_id`, `workspace_slug`, or `external_workspace_id`.

### Request Body

| Field | Type | Required | Description |
|---|---|---|---|
| `workspace_id` | UUID | Conditional | Existing MIOSA workspace ID |
| `workspace_slug` | string | Conditional | Existing workspace slug |
| `external_workspace_id` | string | Conditional | Your external workspace/customer ID |
| `name` | string | Yes | Human-readable project name |
| `slug` | string | No | URL-safe identifier. Auto-derived from `name` if omitted. |
| `external_project_id` | string | No | Your project/app/document ID |
| `description` | string | No | Optional description |
| `metadata` | object | No | Caller metadata stored on the project |

### Example
```bash
curl -X POST https://api.miosa.ai/api/v1/projects \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace_slug": "acme-corp",
    "name": "Lead Magnet",
    "slug": "lead-magnet",
    "external_workspace_id": "acct_123",
    "external_project_id": "project_789"
  }'
```

### Response - `201 Created`

Full project object.

### Errors

| Status | Code | Cause |
|---|---|---|
| 400 | `WORKSPACE_REQUIRED` | No `workspace_id`, `workspace_slug`, or `external_workspace_id` was supplied |
| 404 | `WORKSPACE_NOT_FOUND` | Workspace selector did not resolve inside the organization |
| 422 | `VALIDATION_FAILED` | Name, slug, or external project ID failed validation |

---

## Get a Project

**`GET /api/v1/projects/{id}`**

Returns one project if it belongs to the authenticated organization.

```bash
curl https://api.miosa.ai/api/v1/projects/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Update a Project

**`PATCH /api/v1/projects/{id}`**

### Request Body

| Field | Type | Description |
|---|---|---|
| `name` | string | New display name |
| `slug` | string | New URL-safe slug, unique inside the workspace |
| `description` | string | New description (`null` to clear) |
| `external_project_id` | string | Your project/app/document ID |
| `external_workspace_id` | string | Your customer/workspace ID |
| `metadata` | object | Replacement metadata map |
| `settings` | object | Replacement settings map. |

```bash
curl -X PATCH https://api.miosa.ai/api/v1/projects/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Summer Lead Magnet"}'
```

---

## Use Projects on Resource Create

Most create endpoints accept the same ownership fields:

```json
{
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "project_id": "660e8400-e29b-41d4-a716-446655440001",
  "external_workspace_id": "acct_123",
  "external_project_id": "project_789"
}
```

You can use slugs instead of UUIDs:

```json
{
  "workspace_slug": "acme-corp",
  "workspace_name": "Acme Corp",
  "project_slug": "lead-magnet",
  "project_name": "Lead Magnet"
}
```

This applies to sandboxes, computers, deployments, databases, storage buckets, volumes, edge functions, cron jobs, project auth, and integrations. Derived records inherit ownership automatically.

---

## Domains

Projects do not own a preview domain. Preview domains are an organization
setting, and per-deployment custom domains are attached to the deployment:

- [Custom Domains API](/docs/api-reference/custom-domains) - the organization preview domain and deployment/computer custom domains.

## See also

- [Workspaces API](/docs/api-reference/workspaces) - parent resource
- [Ownership and Attribution](/docs/platform/attribution) - how IDs flow to usage and events
- [Sandboxes API](/docs/api-reference/sandboxes) - create resources inside a project
- [Deployments API](/docs/api-reference/deployments) - publish projects to URLs
