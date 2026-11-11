---
slug: "api-reference/sandbox-shares"
title: "Sandbox Shares API"
description: "Create and manage public share URLs for sandboxes."
section: "API reference"
order: 12027
---

Sandbox shares are publicly accessible URLs that give read-only access to a running sandbox. Unlike preview tokens, share URLs are not scoped to an iframe and do not require the recipient to hold any credentials.

Share tokens use the prefix `ms_` followed by a base64url-encoded payload. The public URL pattern is:

```text
https://{sandbox-id}.sandboxes.miosa.ai?ms=ms_<token>
```

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/sandboxes/{id}/shares` | Create a share URL |
| `GET` | `/api/v1/sandboxes/{id}/shares` | List active shares |
| `DELETE` | `/api/v1/sandboxes/{id}/shares/{share_id}` | Revoke a share |

---

## Create a Share

**`POST /api/v1/sandboxes/{id}/shares`**

Creates a public share link. The `share_url` field is safe to distribute  -  it contains the token as a query parameter.

### Auth

```http
Authorization: Bearer msk_...
```

### Request Body

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `expires_in` | integer | No | `3600` | TTL in seconds |
| `scope` | string | No | `"read"` | Must be `"read"` (only supported value) |

```json
{
  "expires_in": 86400
}
```

### Response  -  `201 Created`

```json
{
  "share_id": "shr_01hx9m7n3k...",
  "share_url": "https://sbx-abc123.sandboxes.miosa.ai?ms=ms_eyJ...",
  "expires_at": "2026-05-27T12:00:00Z",
  "scope": "read"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `share_id` | string | Opaque share identifier used to revoke |
| `share_url` | string | Publicly accessible URL with token embedded |
| `expires_at` | string | ISO 8601 expiry, or `null` for no expiry |
| `scope` | string | Always `"read"` |

---

## List Shares

**`GET /api/v1/sandboxes/{id}/shares`**

Returns all active (non-revoked, non-expired) shares for the sandbox.

### Response  -  `200 OK`

```json
{
  "data": [
    {
      "share_id": "shr_01hx9m7n3k...",
      "share_url": "https://sbx-abc123.sandboxes.miosa.ai?ms=ms_eyJ...",
      "expires_at": "2026-05-27T12:00:00Z",
      "scope": "read"
    }
  ]
}
```

---

## Revoke a Share

**`DELETE /api/v1/sandboxes/{id}/shares/{share_id}`**

Immediately invalidates the share. Any visitor using the share URL will receive a 403 after revocation.

### Response  -  `204 No Content`

No response body.

---

## Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 | `sandbox_not_found` | Sandbox does not exist or wrong tenant |
| 403 | `forbidden` | Tenant mismatch |
| 404 | `not_found` | Share ID does not exist (on DELETE) |
| 422 | `invalid_scope` | `scope` value is not `"read"` |

---

## Examples

**cURL**

```bash
# Create
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 86400}'

# List
curl https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer msk_u_..."

# Revoke
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares/shr_01hx9m7n3k \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Create
share = client.sandboxes.shares.create("sbx_abc123", expires_in=86400)
print(share.share_url)

# List
shares = client.sandboxes.shares.list("sbx_abc123")

# Revoke
client.sandboxes.shares.delete("sbx_abc123", share.share_id)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Create
const share = await client.sandboxes.shares.create('sbx_abc123', {
  expiresIn: 86400,
});
console.log(share.shareUrl);

// List
const shares = await client.sandboxes.shares.list('sbx_abc123');

// Revoke
await client.sandboxes.shares.delete('sbx_abc123', share.shareId);
```
