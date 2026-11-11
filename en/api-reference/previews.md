---
slug: "api-reference/previews"
title: "Previews and publish API"
description: "Expose a sandbox port as a preview URL, publish a sandbox to a stable deployment, and promote a running sandbox to a deployment route."
section: "API reference"
order: 12017
---

Base path: `/api/v1/sandboxes/{id}`

> **Note**
>
> For preview token minting (embedding a sandbox in an iframe without an API key), see the [Preview Tokens API](/docs/api-reference/preview-tokens).

## Expose a Preview Port

**`POST /api/v1/sandboxes/{id}/expose`**

Returns a public, tenant-aware preview URL for a server running inside the sandbox. Use this when an agent generated a Vite/Next/FastAPI/Flask app and started it on a local port.

Inside the sandbox, bind dev servers to `0.0.0.0`, not `localhost`:
```bash
npm run dev -- --host 0.0.0.0 --port 5173
python -m http.server 8000 --bind 0.0.0.0
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `port` | integer | No | Port inside the sandbox to expose. If omitted, MIOSA uses the template lifecycle preview port when available. Must be `1` through `65535` when provided. |

### Response  -  `200 OK`
```json
{
  "url": "https://5173-sbx01j9x.sandbox.miosa.ai"
}
```

If the tenant has a white-label preview domain configured, the same endpoint returns that domain instead of the platform default. Default app ports such as `3000`, `5173`, `8080`, `8000`, and `80` may be returned as `https://{slug}.sandbox.{domain}`; non-default ports use `https://{port}-{slug}.sandbox.{domain}`.

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `MISSING_PARAM` | `port` was omitted and the sandbox template has no default preview port. |
| 409 | `SANDBOX_NOT_RUNNING` | Sandbox is not in `running` state. |
| 422 | `INVALID_PORT` | Port is outside `1` through `65535`. |
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/expose \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"port": 5173}'
```

For static artifacts such as PDFs, images, Markdown, CSV, or ZIP files, write them under `/workspace` and download them through the files API. For web apps, start a server and expose the port.

---

## Publish a Sandbox

**`POST /api/v1/sandboxes/{id}/publish`**

Freezes sandbox source, creates an immutable release and deployment version, verifies the candidate, and promotes it behind a stable deployment URL.
Use this endpoint for production publishing.

The CLI is the recommended happy path:

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name <deployment> \
  --type docker_deploy \
  --wait \
  --json
```

Use the same `--name` on every publish so MIOSA preserves the deployment ID and canonical URL.

See [Publishing](/docs/docker-host/publishing) for request options, proof, and failure behavior.

## Promote a Sandbox Runtime to a Deployment

**`POST /api/v1/sandboxes/{id}/deploy`**

This compatibility bridge routes a persistent deployment URL directly to a running sandbox.
The sandbox remains the runtime behind the route and its auto-destroy timer is cancelled.

Use it only when the sandbox itself must remain the runtime.
For immutable production versions and Docker Host, use `/publish`.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Human-readable deployment name. Used to create the deployment slug. |
| `port` | integer | No | Runtime port to route. Defaults to the template lifecycle preview port, then `80`. |
| `domain` | string | No | Custom domain to attach. |
| `custom_domain` | string | No | Alias for `domain`, used by frontend clients. |

### Response  -  `201 Created`
```json
{
  "deployment_id": "dep_01j9xr2t4fk8me3n5q",
  "url": "https://my-app-a1b2c3.acme.miosa.app",
  "state": "running"
}
```

Managed deployment URLs are tenant-scoped: `https://{deployment-slug}.{tenant-slug}.miosa.app`. MIOSA persists the sandbox runtime target on the deployment and reconciles running routes, so a temporary proxy restart or admin API outage can be repaired from the database.

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `MISSING_PARAM` | `name` was omitted. |
| 409 | `SANDBOX_NOT_RUNNING` | Sandbox is not in `running` state. |
| 409 | `SANDBOX_RUNTIME_UNAVAILABLE` | Sandbox is marked running but has no VM IP yet. |
| 422 | `INVALID_PORT` | Port is outside `1` through `65535`. |
| 422 | `VALIDATION_ERROR` | Deployment record validation failed. |
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/deploy \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"my-app","port":8000}'
```
