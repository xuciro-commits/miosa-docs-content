---
slug: "api-reference/versions"
title: "API Reference / Versions"
description: "Immutable deployment version sub-resource. List, get, promote, archive."
section: "API reference"
order: 12040
---

A **Deployment Version** is the immutable record of one publish. See [Versions](/docs/docker-host/versions) for the conceptual model.

## Endpoints
```http
GET  /api/v1/deployments/:id/versions
GET  /api/v1/deployments/:id/versions/:version_id
POST /api/v1/deployments/:id/versions/:version_id/promote
```

## List
```http
GET /api/v1/deployments/:id/versions
GET /api/v1/deployments/:id/versions?state=ready
GET /api/v1/deployments/:id/versions?workspace_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/deployments/:id/versions?external_workspace_id=acct_123
GET /api/v1/deployments/:id/versions?limit=50
```

Response:
```json
{
  "data": [
    {
      "id": "ver_...",
      "deployment_id": "dep_...",
      "version_number": 17,
      "kind": "static",
      "state": "ready",
      "artifact_sha256": "a5e6f0c1...",
      "source_sha256": "f1a3b8c2...",
      "promoted_at": "2026-05-14T18:20:00Z",
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "project_id": "660e8400-e29b-41d4-a716-446655440001",
      "external_workspace_id": "acct_123",
      "external_user_id": "user-456",
      "created_at": "2026-05-14T18:19:48Z"
    }
  ],
  "total": 17
}
```

## Get
```http
GET /api/v1/deployments/:id/versions/:version_id
```

Returns the full version row including `artifact_manifest`, `runtime_image`, `runtime_command`, `runtime_port`, `health_check_path`, `build_log_uri`.

## Promote

Make a specific ready version the active one for the deployment.
```http
POST /api/v1/deployments/:id/versions/:version_id/promote
```

Body: empty, or optionally `{ "environment": "production" }`.

Scopes: `deployments:write`. Requires `Idempotency-Key` for safety on retries.

Promotion is different from publish:
- **Publish** creates a new version from a sandbox source.
- **Promote** points an existing ready version as active in an environment.

Use promote for canary / staged rollouts: publish to a non-production environment first, validate, then promote to production.

## States

| State | Meaning |
|---|---|
| `created` | Row exists, build not started |
| `building` | Build in progress |
| `ready` | Build succeeded, artifact uploaded, promotable |
| `failed` | Build or health check failed |
| `archived` | Promoted-out-of; still bootable for rollback |

State transitions are one-way. Once `ready` or `failed`, a version stays that way.

## Permissions

| Action | Scope |
|---|---|
| List, Get | `deployments:read` |
| Promote | `deployments:write` |

## See also

- [Deployments](/docs/api-reference/deployments)  -  parent resource
- [Releases](/docs/api-reference/releases)  -  artifact reference
- [Versions](/docs/docker-host/versions)  -  conceptual model
- [Rollback](/docs/docker-host/rollback)  -  uses promote internally
