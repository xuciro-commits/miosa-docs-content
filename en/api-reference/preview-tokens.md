---
slug: "api-reference/preview-tokens"
title: "Preview Tokens API"
description: "Generate short-lived signed tokens for iframe embedding without exposing an API key."
section: "API reference"
order: 12028
---

Preview tokens (`mp_*`) are short-lived signed credentials that allow you to embed a running sandbox in an `<iframe>` from a third-party frontend without exposing your `msk_*` API key to the browser.

Base path: `/api/v1/sandboxes/{id}/preview-token`

> **Note**
>
> Preview tokens are single-resource. Each token is scoped to the sandbox it was minted for. Tokens expire after at most 24 hours.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/sandboxes/{id}/preview-token` | Mint a preview token |

---

## Mint a Preview Token

**`POST /api/v1/sandboxes/{id}/preview-token`**

Generates a signed preview token tied to the sandbox. The returned `url` is ready to use as an iframe `src`.

### Auth

Requires an API key or JWT with the `previews:write` scope.

```http
Authorization: Bearer msk_...
```

### Request Body

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `expires_in` | integer | No | `3600` | Token TTL in seconds (max `86400`) |
| `scope` | string | No | `"read"` | `"read"` or `"interact"` |

```json
{
  "expires_in": 3600,
  "scope": "read"
}
```

### Response  -  `200 OK`

```json
{
  "token": "mp_eyJhbGciOiJIUzI1NiJ9...",
  "url": "https://sbx-abc123.sandbox.miosa.ai?mt=mp_eyJ...",
  "expires_at": "2026-05-26T04:00:00Z",
  "scope": "read"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `token` | string | Signed preview token (`mp_<base64url>`) |
| `url` | string | Iframe-ready URL with token pre-embedded as `?mt=` query param |
| `expires_at` | string | ISO 8601 expiry timestamp |
| `scope` | string | `"read"` or `"interact"` |

### Scopes

| Scope | Description |
|-------|-------------|
| `read` | View-only. Mouse and keyboard input are blocked. |
| `interact` | Full mouse/keyboard input forwarded into the sandbox. |

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 | `sandbox_not_found` | Sandbox does not exist or belongs to a different tenant |
| 403 | `forbidden` | Tenant mismatch |
| 422 | `invalid_scope` | `scope` is not `"read"` or `"interact"` |

---

## Examples

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/preview-token \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 3600, "scope": "read"}'
```

**Python**

```python
import miosa

client = miosa.Miosa()

token = client.sandboxes.preview_token(
    "sbx_abc123",
    expires_in=3600,
    scope="read",
)

print(token.url)  # embed this in your frontend
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

const token = await client.sandboxes.previewToken('sbx_abc123', {
  expiresIn: 3600,
  scope: 'read',
});

// Pass token.url to your frontend  -  safe to expose in the browser
console.log(token.url);
```

---

## Token Format

Preview tokens use the prefix `mp_` followed by a base64url-encoded signed payload. The token is consumed by the sandbox proxy when passed as the `?mt=` query parameter. Do not parse the token body  -  treat it as opaque.

---

## Iframe Embedding

```html
<!-- Read-only embed  -  no API key in the browser -->
<iframe
  src="https://sbx-abc123.sandbox.miosa.ai?mt=mp_eyJ..."
  width="1280"
  height="800"
  allow="clipboard-read; clipboard-write"
></iframe>
```

The token is verified by the sandbox proxy on every request during the token's lifetime. Once it expires, the iframe will show an access-denied page.
