---
slug: "api-reference"
title: "API Reference"
description: "REST API reference for the MIOSA platform - base URL, auth, request/response envelopes, and endpoint guides."
section: "API reference"
order: 12009
---

> **Tip**
>
> Want to browse and try endpoints without writing code?
> The [Interactive Sandbox API Reference](/docs/api) renders the
> allowlisted public Sandbox V1 OpenAPI contract with an in-browser request runner.
> Use the endpoint guides below for the wider platform API.

Every MIOSA API endpoint is REST over HTTPS, returns JSON, and follows a consistent envelope format. The SDKs wrap these endpoints  -  you can use them directly when you need low-level control or want to integrate from a language without an official SDK.

## Base URL
```text

https://api.miosa.ai/api/v1
```

## Authentication

All requests require a Bearer token:
```http
Authorization: Bearer msk_u_...
```

MIOSA API keys (`msk_*`) and short-lived JWT tokens are both accepted. Keys are scoped to an organization and its workspaces/projects. See [API Keys](/docs/platform/api-keys) for scope details.

## Request format

- `Content-Type: application/json` for all request bodies.
- File uploads use `multipart/form-data`.
- All field names are **snake_case**.
- Mutation requests (`POST`, `PUT`, `PATCH`, `DELETE`) should include an `Idempotency-Key` header to enable safe retries:
```http
POST /api/v1/sandboxes
Authorization: Bearer msk_u_...
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "template": "miosa-sandbox",
  "workspace_slug": "acme-corp",
  "project_slug": "lead-magnet",
  "external_workspace_id": "acct_123",
  "external_user_id": "user-456",
  "external_project_id": "project_789"
}
```

Use a UUID v4 as the idempotency key. Repeating the same key within 24 hours returns the original response without re-executing the mutation.
The key also keeps a retry safe: an idempotent create whose placement outcome is uncertain answers a retryable `503 SANDBOX_PLACEMENT_RECONCILING` with `Retry-After` instead of a terminal `409`, so retrying with the same key resolves to the same sandbox.

## Response envelope

Single-resource responses:
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "my-sandbox",
    "status": "running",
    "created_at": "2026-05-17T10:00:00Z"
  }
}
```

List responses include a pagination object:
```json
{
  "data": [ { "id": "...", "name": "..." } ],
  "page": {
    "page": 1,
    "page_size": 20,
    "total": 150
  }
}
```

Pagination parameters accepted by all list endpoints:

| Parameter | Type | Default | Max |
|---|---|---|---|
| `page` | integer | 1 |  -  |
| `page_size` | integer | 20 | 100 |

## Error envelope

All errors follow a single shape:
```json
{
  "error": {
    "code": "sandbox_not_found",
    "message": "No sandbox with id sbx_abc exists in this workspace.",
    "request_id": "req_01jv..."
  }
}
```

Include the `request_id` when contacting support.

### HTTP status codes

| Status | Meaning |
|---|---|
| 200 | Success |
| 201 | Created |
| 202 | Accepted  -  async operation started |
| 400 | Bad request  -  missing or invalid parameters |
| 401 | Unauthorized  -  invalid or missing token |
| 403 | Forbidden  -  valid token, insufficient scope |
| 404 | Not found |
| 409 | Conflict  -  invalid state transition (e.g. starting a running computer) |
| 413 | Payload too large  -  file upload exceeds 10 MB |
| 422 | Unprocessable entity  -  validation error |
| 429 | Rate limited |
| 500 | Internal server error |
| 502 | Bad gateway  -  command service unreachable |

## Rate limits

Rate limits apply per API key, tenant or organization, and endpoint family.
The current self-service defaults are 600 requests per minute for user keys and 30,000 requests per minute for platform keys.
Specific endpoint families and tenant policies can impose lower limits.
Treat `X-RateLimit-*` and `Retry-After` response headers as authoritative.

## IDs and timestamps

Canonical resource IDs returned by the current API are UUIDs.
Some older SDK examples and compatibility surfaces may still show human-readable aliases.
Pass the exact ID returned by the API instead of constructing or prefixing an identifier.
All timestamps are ISO 8601 in UTC: `2026-05-17T10:00:00Z`.

## Ownership fields

Create endpoints for resources accept the same ownership selectors:

| Field | Use |
|---|---|
| `workspace_id` | Existing MIOSA workspace UUID |
| `workspace_slug` | Existing or auto-created workspace slug |
| `workspace_name` | Display name used when auto-creating a workspace |
| `project_id` | Existing MIOSA project UUID |
| `project_slug` | Existing or auto-created project slug |
| `project_name` | Display name used when auto-creating a project |
| `external_workspace_id` | Your customer/account/workspace ID |
| `external_user_id` | Your end-user ID |
| `external_project_id` | Your project/app/document ID |

Responses include `workspace_id` and `project_id` when the resource is owned by a workspace/project. See [Ownership and Attribution](/docs/platform/attribution).

---

## Compute

- **Sandboxes**: Create, list, get, stop, and delete sandbox environments. The core compute primitive.
  [Open →](/docs/api-reference/sandboxes)
- **Agent Runs**: Dispatch prompts into sandboxes or computers through OSA, Codex, Claude Code, Pi, Hermes, or a custom runtime.
  [Open →](/docs/api-reference/agent-runs)
- **Agent Runtime Profiles**: Configure reusable agent runtimes, tools, connector grants, policy, and non-secret env defaults.
  [Open →](/docs/api-reference/agent-runtime-profiles)
- **Exec**: Run bash commands and Python code inside a running computer or sandbox.
  [Open →](/docs/api-reference/exec)
- **Streaming Exec**: Server-sent events stream for long-running commands  -  line-by-line stdout/stderr.
  [Open →](/docs/api-reference/streaming-exec)
- **Files**: Read, write, stat, mkdir, rename, copy, and chmod files inside a computer.
  [Open →](/docs/api-reference/files)
- **Filesystem**: Directory listings, multi-file operations, and archive upload/download.
  [Open →](/docs/api-reference/filesystem)
- **Snapshots**: Create, restore, and delete point-in-time snapshots of a computer's disk state.
  [Open →](/docs/api-reference/snapshots)

## Deploy

- **Deployments**: Manage static and server deployments  -  create, list, get, and delete.
  [Open →](/docs/api-reference/deployments)
- **Versions**: Immutable deployment versions. Each publish creates a new version.
  [Open →](/docs/api-reference/versions)
- **Releases**: Promote a version to a named release (production, staging, etc.).
  [Open →](/docs/api-reference/releases)
- **Custom Domains**: Register, verify, and manage custom domains on deployments.
  [Open →](/docs/api-reference/custom-domains)

## Computers

- **Computers**: Full lifecycle for desktop VM computers  -  create, start, stop, restart, resize.
  [Open →](/docs/api-reference/computers)
- **Desktop**: Desktop control actions: screenshot, click, type, key, scroll, drag, hotkey, windows, accessibility tree.
  [Open →](/docs/api-reference/desktop)
- **OpenComputers**: Register your own hardware as a MIOSA computer (BYOC). One command, any machine.
  [Open →](/docs/api-reference/open-computers)

## Platform

- **Workspaces**: Customer/client workspaces inside an organization.
  [Open →](/docs/api-reference/workspaces)
- **Projects**: Apps, websites, documents, and workflows inside a workspace.
  [Open →](/docs/api-reference/projects)
- **Services**: Managed background services attached to a computer (databases, queues, etc.).
  [Open →](/docs/api-reference/services)
- **Databases**: Provision managed databases and operate lifecycle, backups, restores, branches, replicas, credentials, logs, and metrics.
  [Open →](/docs/api-reference/databases)
- **Network Policy**: Inbound and outbound network rules for a computer.
  [Open →](/docs/api-reference/network-policy)
- **Events**: Lifecycle events emitted by computers and sandboxes  -  for webhooks and audit logs.
  [Open →](/docs/api-reference/events)
- **Credits**: Read credit balance, list usage transactions, and top up.
  [Open →](/docs/api-reference/credits)
- **Regions**: List available regions and their current capacity.
  [Open →](/docs/api-reference/regions)

> **Note**
>
> SDK coverage varies by language and release.
> Use the current SDK page for your language to confirm a helper exists.
> You can always call the documented REST endpoint directly.
