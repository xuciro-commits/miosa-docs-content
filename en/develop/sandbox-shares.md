---
slug: "develop/sandbox-shares"
title: "Sandbox shares"
description: "Create a public, read-only link to a running sandbox so a teammate or an outside user can view it, and revoke the link at any time."
section: "Sandboxes"
order: 2016
---

A **share** is a publicly accessible URL that gives read-only access to a running sandbox. Unlike a preview token, a share URL is not scoped to an iframe and the recipient does not need any credentials: the token is embedded in the URL.

```text
https://{sandbox-id}.sandboxes.miosa.ai?ms=ms_<token>
```

> **Note: Read-only, and revocable**
>
> A share is view-only. The recipient can watch, not drive. Revoke a share and the link stops working immediately, returning `403` to anyone still holding it.

## Create a share

**Python**

```python
share = client.sandboxes.shares.create("sbx_abc123", expires_in=86400)
print(share.share_url)
```

**TypeScript**

```typescript
const share = await client.sandboxes.shares.create('sbx_abc123', { expiresIn: 86400 });
console.log(share.shareUrl);
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 86400}'
```

| Field | Type | Default | Description |
|---|---|---|---|
| `expires_in` | integer | `3600` | TTL in seconds |
| `scope` | string | `"read"` | The only supported value |

The response includes `share_id` (used to revoke), `share_url` (safe to distribute), `expires_at`, and `scope`.

## List and revoke

**Python**

```python
for s in client.sandboxes.shares.list("sbx_abc123"):
    print(s.share_id, s.expires_at)

client.sandboxes.shares.delete("sbx_abc123", share.share_id)
```

**curl**

```bash
curl https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares/shr_01hx9m7n3k \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Listing returns only active shares  -  non-revoked and not yet expired. Revoking returns `204 No Content`.

## Shares, previews, and browser tokens

Three different ways to hand someone access, for three different jobs:

| Mechanism | Access | Use it to |
|---|---|---|
| Sandbox share | Read-only view of a running sandbox | Let someone watch a sandbox from a normal link |
| Preview token | HTTP access to one exposed port | Embed a running app in an iframe |
| Browser access token | A live desktop stream | Embed a desktop for one end-user |

See [Previews](/docs/develop/previews) for ports and [Browser access tokens](/docs/platform/browser-tokens) for desktop embeds.

- [Sandbox shares API](/docs/api-reference/sandbox-shares): Wire format, error codes, and every endpoint.
- [Previews](/docs/develop/previews): Expose a port as a live HTTPS URL, with visibility and token controls.
