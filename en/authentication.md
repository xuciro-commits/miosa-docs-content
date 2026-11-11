---
slug: "authentication"
title: "Authentication"
description: "How to authenticate with the MIOSA API using msk_ API keys or JWT tokens."
section: "API reference"
order: 12000
---

Every MIOSA API request carries a single `Authorization: Bearer &lt;token&gt;` header.
The token is either an **API key** (prefix `msk_`) or a **JWT** from `POST /auth/login`.

One key covers the entire platform  -  computers, desktop, files, exec, CUA agent,
OSA, credits, and (with the right role) admin. No separate keys for AI, no
per-product configuration.

## Credential hierarchy

```mermaid
graph TB
    MSK[msk_* workspace key] -->|server-side only| API[MIOSA API calls]
    MSK -->|mints via POST /api/v1/browser-tokens| BT[Browser token - scoped + short-lived]
    BT -->|browser only| EP[Specific sandbox endpoint]
    MSK -->|mints via POST /api/v1/auth/sse-ticket| ST[SSE ticket - single use]
    ST -->|EventSource| SS[SSE stream]
    MSK -->|mints via POST /api/v1/sandboxes/:id/preview-token| MP[mp_ Preview token]
    MP -->|iframe embed - no msk_ exposed| Preview[MiosaPreview component]
    MSK -->|mints via POST /api/v1/sandboxes/:id/shares| MS[ms_ Share token]
    MS -->|URL param - public read-only| ShareURL[Public share URL]
```

## API Keys

### Key format
```text

msk_<role>_<random>
```

| Prefix | Role | Capabilities |
|--------|----------|---------------------------------------------------|
| `msk_u_...` | user | Compute, desktop, files, CUA, OSA, credits |
| `msk_a_...` | admin | Everything above **+** `/api/v1/admin/*` |
| `msk_p_...` | platform | Tenant-wide automation issued by the MIOSA platform |

The public self-service endpoint creates normal API-purpose keys.
Specialized operator or intelligence credentials use separate controlled flows.

### Creating an API key
```bash
curl -X POST https://api.miosa.ai/api/v1/api-keys \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production SDK Key",
    "key_type": "user"
  }'
```
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "key": "msk_u_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
    "key_prefix": "msk_u_a1b2c3",
    "name": "Production SDK Key",
    "key_type": "user",
    "key_purpose": "api",
    "rate_limit_rpm": 600,
    "status": "active",
    "created_at": "2026-04-17T00:00:00Z"
  }
}
```

> **Caution**
>
> The raw key is **only returned once** at creation. Store it securely. If lost,
> revoke it and create a new one  -  the original cannot be recovered.

Users can mint personal `user` keys.
Tenant owners and administrators can also mint `platform` keys for backend automation.
Admin keys remain operator-internal and are not created through the public self-service endpoint.

### Using an API key

Pass the key in the `Authorization` header:
```bash
curl https://api.miosa.ai/api/v1/computers \
  -H "Authorization: Bearer msk_u_a1b2c3d4e5f6..."
```

All five SDKs read `MIOSA_API_KEY` from the environment by default:

**Python**

```python
  from miosa import Miosa

  # Explicit key
  client = Miosa(api_key="msk_u_...")

  # Or from environment: export MIOSA_API_KEY="msk_u_..."
  client = Miosa()
```

**TypeScript**

```ts
  import { Miosa } from '@miosa/sdk';

  // Explicit key
  const miosa = new Miosa({ apiKey: 'msk_u_...' });
```

**Go**

```go
  import "github.com/miosa-ai/miosa-go"

  // Explicit key
  client := miosa.NewClient("msk_u_...")

  // Or from environment
  client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))
```

**Elixir**

```elixir
  # Explicit key
  client = Miosa.client("msk_u_...")

  # Or from environment
  client = Miosa.client(System.fetch_env!("MIOSA_API_KEY"))
```

**Java**

```java
  import ai.miosa.sdk.MiosaClient;

  MiosaClient miosa = new MiosaClient("msk_u_...");
```

### Managing keys

| Action | Endpoint |
|--------|----------|
| List your keys | `GET /api/v1/api-keys` |
| Create a key | `POST /api/v1/api-keys` |
| Revoke a key | `DELETE /api/v1/api-keys/{id}` |

Revoking a key flips its status to `revoked`. The change is cached and
enforced within seconds  -  subsequent requests return `401 Unauthorized`.
Revocation is irreversible; create a new key if you need access restored.

### Admin keys

`msk_a_*` keys unlock operator-admin endpoints.
`msk_p_*` keys are trusted tenant automation credentials, but they do not automatically make a caller a MIOSA operator.

User-role callers hitting an admin endpoint receive `403 Forbidden`.

Admin endpoints are operator-internal and not documented publicly.

## Preview tokens

Preview tokens (`mp_<base64url>`) are short-lived signed tokens for embedding a sandbox preview in an `<iframe>` without exposing an `msk_*` key to the browser.

Your backend mints a preview token server-side and passes it to the `<MiosaPreview>` UI component. The component includes it automatically  -  no `Authorization` header needed in browser code.

### Minting a preview token

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/preview-token \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 3600}'
```
```json
{
  "data": {
    "token": "mp_eyJhbGciOiJFZERTQSIs...",
    "expires_at": "2026-05-25T13:00:00Z",
    "sandbox_id": "sbx_abc123"
  }
}
```

**Python**

```python
from miosa import Miosa

client = Miosa(api_key="msk_u_...")

preview = client.sandboxes.create_preview_token(
    sandbox_id="sbx_abc123",
    expires_in=3600,
)
# Pass preview.token to the browser  -  never the msk_ key.
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const miosa = new Miosa({ apiKey: 'msk_u_...' });

const preview = await miosa.sandboxes.createPreviewToken('sbx_abc123', {
  expiresIn: 3600,
});
// Pass preview.token to the browser  -  never the msk_ key.
```

In the browser, pass the token to the `<MiosaPreview>` component:
```html
<MiosaPreview token="mp_eyJhbGciOiJFZERTQSIs..." sandboxId="sbx_abc123" />
```

See [`<MiosaPreview>` and other UI components](/docs/ui-components) and the [Preview Tokens API reference](/docs/api-reference/preview-tokens) for the full component API and token validation spec.

- **Format:** `mp_<base64url>`  -  signed, not opaque; do not attempt to decode.
- **TTL:** Default 1 hour; configurable via `expires_in` (max 24 hours).
- **Scope:** Read-only access to the specific sandbox's preview port. Cannot create or mutate resources.
- **Revocation:** Preview tokens cannot be individually revoked. Destroying the sandbox invalidates all outstanding tokens for it.

---

## Share tokens

Share tokens (`ms_<base64url>`) turn a sandbox preview into a public read-only URL that anyone with the link can visit  -  no MIOSA account required.

### Minting a share URL

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 86400}'
```
```json
{
  "data": {
    "token": "ms_eyJhbGciOiJFZERTQSIs...",
    "url": "https://sbx-abc123.sandboxes.miosa.ai?ms=ms_eyJhbGciOiJFZERTQSIs...",
    "expires_at": "2026-05-26T00:00:00Z",
    "sandbox_id": "sbx_abc123"
  }
}
```

**Python**

```python
from miosa import Miosa

client = Miosa(api_key="msk_u_...")

share = client.sandboxes.create_share(
    sandbox_id="sbx_abc123",
    expires_in=86400,
)
print(share.url)  # Hand this URL to the end user.
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const miosa = new Miosa({ apiKey: 'msk_u_...' });

const share = await miosa.sandboxes.createShare('sbx_abc123', {
  expiresIn: 86400,
});
console.log(share.url); // Hand this URL to the end user.
```

- **Format:** `ms_<base64url>`  -  the token IS the URL query parameter (`?ms=ms_...`).
- **Access:** Public, read-only, unauthenticated  -  anyone with the URL can view the preview.
- **TTL:** Set via `expires_in` in seconds. No maximum enforced; omit for a non-expiring share.
- **Revocation:** `DELETE /api/v1/sandboxes/:id/shares/:share_id` revokes immediately.

See the [Sandbox Shares API reference](/docs/api-reference/sandbox-shares) and [Integrations: Preview Tokens](/docs/develop/previews).

---

## Token comparison

| Token | Scope | TTL | Who uses it |
|---|---|---|---|
| `msk_*` API key | Full workspace | Until revoked | Server |
| JWT | User session | ~1 h (refresh 7 d) | Server / browser (UI) |
| Browser token | Single sandbox | Configurable (default 1 h) | Browser |
| `mp_` Preview token | Single sandbox preview | Configurable (default 1 h) | Browser / iframe |
| `ms_` Share token | Single sandbox preview | Configurable | Anyone (unauthenticated) |

| Token | How to mint | Can revoke | Creates resources |
|---|---|---|---|
| `msk_*` API key | Dashboard or `POST /api/v1/api-keys` | Yes | Yes |
| JWT | `POST /api/v1/auth/login` | No (short TTL) | Yes |
| Browser token | `POST /api/v1/browser-tokens` | No (short TTL) | No |
| `mp_` Preview token | `POST /api/v1/sandboxes/:id/preview-token` | No | No |
| `ms_` Share token | `POST /api/v1/sandboxes/:id/shares` | Yes | No |

---

## White-label auth note

The `tenant.preview_domain.*` and `tenant.branding.*` configuration endpoints accept **both** `msk_*` API keys and JWTs. These are the only tenant-level config routes accessible with an API key  -  all other tenant administration routes require a JWT from an admin session.

```bash
# Setting a preview domain with an API key (allowed)
curl -X PUT https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_a_..." \
  -H "Content-Type: application/json" \
  -d '{"domain": "preview.yourdomain.com"}'

# Fetching tenant branding with an API key (allowed)
curl https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_a_..."
```

See the [Tenant Preview Domain](/docs/api-reference/tenant-preview-domain) and [Tenant Branding](/docs/api-reference/tenant-branding) API references for the full configuration spec.

---

## JWT Tokens

JWT tokens are used by the MIOSA web UI and SSE ticket flows. They are issued
on login and carry user identity + tenant context.

### Obtaining a token
```bash
curl -X POST https://api.miosa.ai/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "your_password"}'
```
```json
{
  "token": "eyJhbGciOiJIUzUxMiIs...",
  "refresh_token": "eyJhbGciOiJIUzUxMiIs...",
  "user": { "id": "...", "email": "user@example.com" }
}
```

### Token lifecycle

| Token | Lifetime | Refresh |
|-------|----------|---------|
| Access token | ~1 hour | Use refresh token |
| Refresh token | 7 days | Re-login required |
```bash
curl -X POST https://api.miosa.ai/api/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{"refresh_token": "eyJhbGciOiJIUzUxMiIs..."}'
```

> **Note**
>
> For programmatic access, use **API keys**. JWTs are designed for browser
> sessions and expire frequently.

## SSE streams

Server-Sent Event endpoints (CUA session events) cannot use a standard
`Authorization` header because `EventSource` does not support custom headers.
Instead, mint a one-time ticket first:
```bash
# 1. Ask for a ticket (Bearer-authed  -  API key or JWT both work)
curl -X POST \
  https://api.miosa.ai/api/v1/computers/{id}/cua/sessions/{session_id}/sse-ticket \
  -H "Authorization: Bearer msk_u_..."

# {"ticket":"sset_abc123","expires_in":3600,"session_id":"..."}

# 2. Open the stream with the ticket as a query param
curl https://api.miosa.ai/api/v1/computers/{id}/cua/sessions/{session_id}/events?ticket=sset_abc123
```

The ticket is single-use and expires after one hour.

## Rate limits

| Scope | Current default | Notes |
|-------|-------|--------|
| User API key | 600 requests per minute | Tenant and endpoint-family policy can lower it |
| Platform API key | 30,000 requests per minute | Restricted to tenant owners and administrators |
| Public auth endpoints | 60 requests per minute per IP | Individual auth actions may have separate controls |

When rate limited, the API returns `429 Too Many Requests` with a
`Retry-After` header.

Rate-limit status is surfaced in every response:
```http

X-RateLimit-Limit: 600
X-RateLimit-Remaining: 587
X-RateLimit-Reset: 1712700060
```

## Error codes

| Status | Code | Meaning |
|--------|------|---------|
| 401 | `UNAUTHORIZED` | Missing / invalid / revoked credential |
| 403 | `FORBIDDEN` | Valid credential but insufficient role |
| 429 | `RATE_LIMITED` | Too many requests |

## Security best practices

1. **Never commit API keys** to version control.
2. **Use environment variables** or a secrets manager.
3. **Rotate keys** on exposure  -  revoke and create a new one.
4. **Scope by role**  -  hand out `msk_u_*` keys to most callers; reserve
   `msk_a_*` / `msk_p_*` for operational tooling.
5. **Separate keys for dev and prod**  -  easier to revoke blast radius.
6. **Restrict by IP** (admin keys)  -  pass `allowed_ips` when minting via
   `POST /admin/api-keys` to reject traffic from unexpected sources.
