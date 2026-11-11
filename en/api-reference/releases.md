---
slug: "api-reference/releases"
title: "Releases API"
description: "Inspect and activate exact immutable deployment artifacts."
section: "API reference"
order: 12041
---

A **Release** is the immutable artifact produced from a build or sandbox publish.
It records the artifact digest, runtime command, port, health check, source attribution, and readiness state used for an exact promotion.

Base path: `/api/v1/deployments/{deployment_id}/releases`

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/deployments/{deployment_id}/releases` | List releases for one deployment |
| `GET` | `/deployments/{deployment_id}/releases/{release_id}` | Inspect one exact release |
| `POST` | `/deployments/{deployment_id}/releases/{release_id}/promote` | Activate one exact release |

Publish creates the candidate release.
Promotion changes the active production artifact only after the requested release is ready and the configured runtime accepts it.

## List releases

```bash
curl https://api.miosa.ai/api/v1/deployments/$DEPLOYMENT_ID/releases \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

The response contains `data`, a compatibility `releases` array, and `total`.

```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "deployment_id": "660e8400-e29b-41d4-a716-446655440000",
      "deployment_version_id": "770e8400-e29b-41d4-a716-446655440000",
      "kind": "dynamic",
      "state": "ready",
      "artifact_sha256": "a5e6f0c1...",
      "start_command": "node server.js",
      "port": 3000,
      "health_check_path": "/api/health",
      "ready_at": "2026-07-24T18:19:48Z",
      "created_at": "2026-07-24T18:18:06Z"
    }
  ],
  "total": 1
}
```

## Inspect one release

```bash
curl \
  https://api.miosa.ai/api/v1/deployments/$DEPLOYMENT_ID/releases/$RELEASE_ID \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Use this endpoint to confirm the exact artifact digest, runtime contract, source sandbox, build, and readiness state before promotion.

## Promote one exact release

Promotion is a mutation and requires an idempotency key.

```bash
curl -X POST \
  https://api.miosa.ai/api/v1/deployments/$DEPLOYMENT_ID/releases/$RELEASE_ID/promote \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Idempotency-Key: $IDEMPOTENCY_KEY"
```

The response contains the deployment plus the durable operation record.
Repeating the same request with the same idempotency key returns the existing result instead of activating the release twice.

> **Note: Runtime-aware activation**
>
> Promotion uses the deployment's configured runtime product.
> For Docker Host, the release is activated through the workspace Docker Host host.
> For standard MIOSA Deploy, it is activated through the standard dynamic or static runtime.

## CLI

```bash
miosa deploy releases <deployment-id> --json
miosa deploy versions <deployment-id> --json
miosa deploy promote <deployment-id> <release-id> --json
miosa deploy rollback <deployment-id> --to <release-id> --yes --json
```

## Immutability

A release is content-addressed by `artifact_sha256`.
Changing application content creates a new release.
Promotion never mutates the bytes of an existing release.

Rollback selects a previously ready release and activates that exact artifact through the same runtime-aware path.

## Permissions

| Action | Required scope |
|---|---|
| List or inspect releases | `deployments:read` |
| Promote or roll back | `deployments:write` |

## See also

- [Publishing](/docs/docker-host/publishing): Create candidate releases from sandbox source.
- [Versions](/docs/api-reference/versions): Inspect logical deployment versions and their build state.
- [Rollback](/docs/docker-host/rollback): Restore a previously healthy release.
