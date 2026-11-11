---
slug: "fundamentals/authentication"
title: "Authentication"
description: "API keys, browser tokens, and the five MIOSA token types with their prefixes, callers, and trust levels."
section: "API reference"
order: 12002
---

Every API request carries one header:
```http
Authorization: Bearer <token>
```

The token is either an **API key** or a **browser token** (scoped JWT).

## API keys

API keys have the prefix `msk_` followed by a role segment:

| Prefix | Role | Use |
|---|---|---|
| `msk_u_...` | user | General-purpose server-side calls |
| `msk_a_...` | admin | Operator/platform tooling |
| `msk_p_...` | platform | Tenant-wide automation |

**API keys are server-side only.** Never put an `msk_*` key in browser code, a mobile app, or a public repository.

> **Caution: Key exposure**
>
> A leaked `msk_*` key grants full workspace access. Revoke it immediately at `DELETE /api/v1/api-keys/{id}` and rotate.

## Browser tokens

For browser-side access  -  embedding a preview, opening a terminal in an iframe  -  your backend mints a short-lived scoped token and passes it to the client:

**Python**

```python
import miosa

client = miosa.Miosa(api_key="msk_u_...")

token = client.browser_tokens.create(
    scope="sandbox",
    resource_id="sbx_abc123",
    expires_in=3600,
)
# Pass token.value to the browser  -  never the msk_ key.
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const miosa = new Miosa({ apiKey: 'msk_u_...' });

const token = await miosa.browserTokens.create({
  scope: 'sandbox',
  resourceId: 'sbx_abc123',
  expiresIn: 3600,
});
// Pass token.value to the browser  -  never the msk_ key.
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/browser-tokens \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "scope": "sandbox",
    "resource_id": "sbx_abc123",
    "expires_in": 3600
  }'
```

Browser tokens expire, are non-renewable, and are scoped to a single resource. They cannot create new resources.

See [Authentication](/docs/authentication) and [Browser Tokens](/docs/platform/browser-tokens) for the full reference.

## Token types

MIOSA issues five distinct token types. Each covers a different caller and trust level:

| Token | Prefix | Caller | TTL | Read / Write |
|---|---|---|---|---|
| API key | `msk_*` | Your server | Until revoked | Read + Write |
| JWT | `ey...` | Admin UI / browser session | ~1 h (refresh 7 d) | Read + Write |
| Browser token | (opaque) | Your user's browser | Configurable (default 1 h) | Read-only, single resource |
| Preview token | `mp_*` | Browser / `<MiosaPreview>` iframe | Configurable (default 1 h) | Read-only, sandbox preview |
| Share token | `ms_*` | Anyone  -  no account required | Configurable | Read-only, sandbox preview |

The `msk_*` API key is the root credential. All other token types are minted downstream from it by your server. Never give an `msk_*` key to a browser or end user.

For minting flows, revocation, and validation rules see [Authentication](/docs/authentication) and [Preview Tokens](/docs/develop/previews).

- [Authentication](/docs/authentication): API key formats, browser tokens, SSE tickets, and rate limit details.
