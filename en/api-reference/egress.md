---
slug: "api-reference/egress"
title: "Egress (Security) API"
description: "API reference for the MIOSA Egress credential vault, network allowlist, OAuth Connect, and audit log."
section: "API reference"
order: 12055
---

The Egress API powers the **Security** tab on every Computer and Sandbox: encrypted credential vault, OAuth Connect, network allowlist with audit-only / enforce modes, and a live-tail audit log.

Base path: `/api/v1/egress`

> **Note**
>
> All endpoints require `Authorization: Bearer &lt;msk_*&gt;`. See [Authentication](/docs/authentication). For higher-level access, use the [SDKs](/docs/sdks/egress).

For the architecture overview, walkthroughs, and white-label patterns, see the [Security guides](/docs/security).

---

## Secrets

### Create a secret

**`POST /api/v1/egress/secrets`**

Stores an encrypted credential and optionally binds it to a resource as an environment variable. The raw value is encrypted with ChaCha20-Poly1305 and never returned again.

```bash
curl -X POST https://api.miosa.ai/api/v1/egress/secrets \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "openai_key",
    "value": "sk-proj-...",
    "type": "api_key",
    "scope": "user",
    "expose_as_env": "OPENAI_API_KEY"
  }'
```

Required body fields: `name`, `value`, `type`, `scope`.

| Field | Values | Notes |
|---|---|---|
| `type` | `api_key`, `oauth_connect`, `oauth_machine` | OAuth flows are usually initiated via `/oauth/start` instead. |
| `scope` | `tenant`, `workspace`, `user`, `external_user`, `external_workspace` | The 5-tier scoping model. |
| `expose_as_env` | shell-safe upper-snake-case | If set, auto-creates a binding. |

Optional scoping fields based on `scope`: `workspace_id`, `owner_user_id`, `external_user_id`, `external_workspace_id`.

Returns `201 Created` with `{data: <public_secret>, binding: <binding_or_null>}`. The `data.preview` is the first-6 + last-4 of the value; the full value is never returned.

### List secrets

**`GET /api/v1/egress/secrets`**

Query params: `scope`, `workspace_id`, `owner_user_id`, `external_user_id`, `external_workspace_id`.

Returns metadata only (no values).

### Get a secret

**`GET /api/v1/egress/secrets/:id`**  -  metadata for one secret.

### Rotate

**`PATCH /api/v1/egress/secrets/:id`**

Re-encrypts with a new value. The bound `placeholder_token` does not change; every sandbox using it picks up the new value on its next outbound request.

```json
{ "value": "sk-proj-new-value", "expires_at": "2026-12-31T00:00:00Z" }
```

### Delete

**`DELETE /api/v1/egress/secrets/:id`**

---

## Bindings

A binding links a secret to a Computer or Sandbox and surfaces it as an environment variable (or opaque header token).

### Create a binding

**`POST /api/v1/egress/bindings`**

```json
{
  "secret_id": "<uuid>",
  "resource_id": "<computer-or-sandbox-id>",
  "resource_type": "sandbox",
  "exposure": "env_var",
  "expose_as_env": "OPENAI_API_KEY"
}
```

Returns the binding including its `placeholder_token`  -  the opaque `miosa-tok-…` value the workload sees in env.

### List + delete

- `GET /api/v1/egress/bindings?resource_id=<id>&resource_type=sandbox`
- `DELETE /api/v1/egress/bindings/:id`

---

## OAuth Connect

### List provider catalog

**`GET /api/v1/egress/oauth/providers`**  -  returns providers visible to the calling tenant (platform defaults merged with tenant overrides). No client secrets exposed.

### Start an OAuth flow

**`POST /api/v1/egress/oauth/start`**

```json
{
  "provider": "github",
  "expose_as_env": "GITHUB_TOKEN",
  "scope": "user",
  "owner_user_id": "<uuid>"
}
```

Returns `{authorize_url, state}`. Open `authorize_url` in a browser; the user approves; the upstream redirects to `/api/v1/egress/oauth/callback?code=&state=`. On success the tokens are encrypted into a new `oauth_connect` secret + binding.

### Status

**`GET /api/v1/egress/oauth/status?state=<state>`**  -  poll while the user is completing the flow. Returns `pending`, `complete`, or `failed`.

### Admin: tenant-override providers (white-label)

- `POST /api/v1/egress/oauth/admin/providers`  -  register your own OAuth app credentials so end-users see your brand on the consent screen.
- `DELETE /api/v1/egress/oauth/admin/providers/:id`

Requires admin (`msk_a_*`) credentials.

---

## Network policies + allowlist

### Policies

- `GET /api/v1/egress/policies`  -  list (optionally filter by `resource_id`)
- `POST /api/v1/egress/policies`  -  create
- `GET /api/v1/egress/policies/:id`
- `PATCH /api/v1/egress/policies/:id`  -  set `mode` (`audit_only` / `enforce`), `default_action` (`allow` / `deny`)

Each Computer or Sandbox resolves the policy in this order: resource override → tenant default. Tenant default starts in `audit_only` mode.

### Allowlist rules

- `GET /api/v1/egress/allowlist?policy_id=<uuid>`
- `POST /api/v1/egress/allowlist`
- `DELETE /api/v1/egress/allowlist/:id`

```json
{
  "policy_id": "<uuid>",
  "pattern": "*.openai.com",
  "kind": "wildcard",
  "method": "POST",
  "path_glob": "/v1/*",
  "action": "allow",
  "warn_only": false,
  "priority": 100
}
```

Pattern kinds: `exact` (literal host), `wildcard` (`*.example.com`), `cidr` (IP literal targets only).

---

## Audit log

### Query

**`GET /api/v1/egress/audit`**

Query params: `resource_id`, `host`, `action` (`allow`/`reject`/`stub`/`error`), `since` (relative like `1h` / `7d` or ISO timestamp), `until`, `limit` (default 100, max 1000), `external_user_id`, `external_workspace_id`.

Each row includes the full identity chain (`tenant_id`, `workspace_id`, `owner_user_id`, `external_user_id`, `external_workspace_id`), request metadata (`host`, `method`, `path`, `sni`, `mode`, `remote_addr`), outcome (`action`, `rejected_by`, `status_code`, `duration_ms`, `bytes_in`, `bytes_out`, `error_message`), and the transform trace JSONB (`request_transforms`, `response_transforms`, `mcp`).

### Get a single event

**`GET /api/v1/egress/audit/:id`**

### Allowlist suggestions

**`GET /api/v1/egress/audit/suggestions?resource_id=<id>`**  -  returns the hosts a resource has contacted recently, ranked by frequency. The basis for the one-click "Lock down with selected as allowlist" UX.

### Live tail (WebSocket)

**`wss://api.miosa.ai/api/v1/egress/audit/resource/:resource_id?token=<jwt>`**

Subprotocol: `miosa-egress-audit-v1`. Intended to push `{"type":"audit_event","data":{...}}` frames in real time as outbound requests are recorded. Egress audit capture is not yet enabled in the current release, so no frames are emitted yet.

Tenant-scoped variant (admin only): `wss://api.miosa.ai/api/v1/egress/audit/tenant/:tenant_id?token=<jwt>`.

---

## Rate limits + retention

| Scope | Limit |
|---|---|
| All `/egress/*` endpoints | Current limit is returned through `X-RateLimit-*` headers |
| Audit retention | 90 days for every tenant; longer on enterprise |
| Audit live-tail (WebSocket) | unlimited; does not count against REST quota |

---

## Errors

Same envelope as the rest of the API  -  `{"error": "...", "details": ...}` with the appropriate status code.

| Status | When |
|---|---|
| `400` | invalid body, unknown scope, malformed env var name |
| `401` | missing / invalid credential |
| `403` | role does not allow the operation (e.g., creating tenant-scoped secret as a non-admin) |
| `404` | secret / binding / policy / event not found in this tenant |
| `409` | secret name collision within scope (unique partial index) |
| `422` | scope ↔ identity column mismatch (e.g., scope=user without owner_user_id) |

---

## SDK shortcut

All endpoints above are wrapped by the [Egress SDK reference](/docs/sdks/egress) with idiomatic methods in Python, TypeScript, Go, Java, and Elixir.
