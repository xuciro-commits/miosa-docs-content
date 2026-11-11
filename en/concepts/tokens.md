---
slug: "concepts/tokens"
title: "Preview and share tokens"
description: "Short-lived preview tokens (mp_*) and revocable share tokens (ms_*) for loading a sandbox preview in a browser without exposing a workspace key."
section: "Optional"
order: 17012
---

## Preview tokens

A **preview token** (prefix `mp_*`) is a short-lived signed token that authorizes a browser to load a sandbox preview without receiving the workspace API key.

The typical flow:

1. Your backend calls `POST /api/v1/sandboxes/{id}/preview-token` with the sandbox ID and a TTL (max 24 hours).
2. MIOSA returns an iframe-ready URL with the token embedded.
3. The browser loads that URL, for example `https://<slug>.sandbox.miosa.ai?mt=mp_...`.
4. MIOSA's edge verifies the token server-side and proxies the request to the sandbox port. The token value never grants broader access.

Preview tokens are non-renewable and non-transferable. When the TTL expires the URL returns `401`. Your backend mints a new token to restore access.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/previews/$PREVIEW/share \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -d '{"ttl_seconds": 3600}'
```

[Browser Tokens reference →](/docs/platform/browser-tokens)

## Share tokens

A **share token** (prefix `ms_*`) is a revocable public share URL for a sandbox preview  -  analogous to a "share link" in a document editor.

Key differences from preview tokens:

| | Preview token (`mp_*`) | Share token (`ms_*`) |
|--|------------------------|----------------------|
| Intended recipient | Your authenticated user | Anyone with the link |
| Minted by | Your backend | Your backend |
| Revocable | Yes (delete the token) | Yes (delete the token) |
| Password protection | No | Optional |
| Expiry | Required TTL | Optional |

Share tokens are useful for demos, client reviews, or collaborative previews where the recipient does not have a user account in your system. The link can be embedded as a plain URL  -  no `Authorization` header needed.

[Previews reference →](/docs/develop/previews)

## Next

- [Tenant configuration](/docs/concepts/tenant-config): preview domains and error-page branding.
- [Browser Tokens reference](/docs/platform/browser-tokens): minting, scopes, and revocation.
- [Core Concepts](/docs/concepts): the one-screen overview.
