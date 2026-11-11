---
slug: "api-reference/deployments"
title: "API Reference / Deployments"
description: "REST API for the Deployment resource - create, list, get, update, delete, publish, rollback."
section: "API reference"
order: 12038
---

A **Deployment** is the stable production object for a published app/site/API. See [Deploy / Overview](/docs/docker-host/overview) for the conceptual model.

> **Note: Recommended publish path**
>
> Use `POST /api/v1/sandboxes/:id/publish` or `miosa deploy create` to turn a tested sandbox into an immutable release.
> Pass an existing deployment ID when updating so its identity and URL remain stable.

## Endpoints
```http
POST   /api/v1/deployments
GET    /api/v1/deployments
GET    /api/v1/deployments/:id
PATCH  /api/v1/deployments/:id
DELETE /api/v1/deployments/:id
POST   /api/v1/deployments/:id/publish
POST   /api/v1/deployments/:id/rollback
POST   /api/v1/sandboxes/:id/publish
POST   /api/v1/sandboxes/:id/deploy                    # Compatibility bridge
```

All endpoints require `Authorization: Bearer &lt;msk_*&gt;`. Mutations should include an `Idempotency-Key` header.

## Create
```http
POST /api/v1/deployments
```

Body:
```json
{
  "name": "Team Portal",
  "source_type": "sandbox",
  "workspace_slug": "acme-corp",
  "workspace_name": "Acme Corp",
  "project_slug": "team-portal",
  "project_name": "Team Portal",
  "external_workspace_id": "acct_123",
  "external_user_id": "user-456",
  "external_project_id": "project_789",
  "metadata": { }
}
```

Ownership fields accepted on create: `workspace_id`, `workspace_slug`, `workspace_name`, `project_id`, `project_slug`, `project_name`, `external_workspace_id`, `external_user_id`, and `external_project_id`.

Scopes: `deployments:write`.

## Publish
```http
POST /api/v1/deployments/:id/publish
```

Body:
```json
{
  "source_sandbox_id": "sbx_...",
  "kind": "auto",
  "environment": "production",
  "output_path": "/workspace",
  "build_command": "npm run build",
  "run_command": "npm start",
  "port": 3000,
  "health_check_path": "/_health",
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "project_id": "660e8400-e29b-41d4-a716-446655440001",
  "external_workspace_id": "acct_123"
}
```

Server-side behavior is documented in [Publishing](/docs/docker-host/publishing).

Response:
```json
{
  "data": {
    "deployment": {
      "id": "dep_...",
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "project_id": "660e8400-e29b-41d4-a716-446655440001",
      "active_version_id": "ver_...",
      "state": "running",
      "public_url": "https://team-portal.example-org.miosa.app"
    },
    "version": {
      "id": "ver_...",
      "kind": "static",
      "state": "ready",
      "artifact_uri": "s3://...",
      "artifact_sha256": "...",
      "source_sha256": "..."
    },
    "release": {
      "id": "rel_...",
      "state": "ready"
    },
    "promoted": true
  }
}
```

## List
```http
GET /api/v1/deployments
GET /api/v1/deployments?workspace_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/deployments?project_id=660e8400-e29b-41d4-a716-446655440001
GET /api/v1/deployments?external_workspace_id=acct_123
GET /api/v1/deployments?external_user_id=user-456
GET /api/v1/deployments?state=running
```

All filters are organization-scoped server-side. See [Ownership and Attribution](/docs/platform/attribution).

## Get
```http
GET /api/v1/deployments/:id
```

Returns the full deployment row including `workspace_id`, `project_id`, `active_version_id`, attribution, and metadata.

## Update
```http
PATCH /api/v1/deployments/:id
```

Body: any subset of mutable fields (`name`, `auto_deploy`, `metadata`, `external_*`).

## Rollback
```http
POST /api/v1/deployments/:id/rollback
```

Body:
```json
{
  "version_id": "ver_..."
}
```

See [Rollback](/docs/docker-host/rollback).

## Delete
```http
DELETE /api/v1/deployments/:id
```

Tears down runtime instances, removes routes, marks the deployment deleted. Versions and releases are retained per the retention policy.

## See also

- [Versions](/docs/api-reference/versions)  -  version sub-resource
- [Releases](/docs/api-reference/releases)  -  build artifact reference
- [Custom Domains](/docs/api-reference/custom-domains)  -  domain sub-resource
- [Deploy / Overview](/docs/docker-host/overview)  -  conceptual model
