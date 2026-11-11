---
slug: "concepts/tenant-config"
title: "Tenant configuration"
description: "Tenant preview domain and tenant branding - the apex hostname that replaces the MIOSA fallback preview domain and the error pages your end users see."
section: "Optional"
order: 17010
---

## Tenant preview domain

A **tenant preview domain** is an apex hostname that replaces the MIOSA fallback preview domain in every sandbox preview URL your end users see. It is set per tenant.

When you set and verify a preview domain (for example, `yourproduct.com`), sandbox previews for your tenant are served at:

```text
https://<sandbox-slug>.yourproduct.com
```

Setup is a two-step DNS verification: add a wildcard CNAME (`*.yourproduct.com`) pointing to `proxy.miosa.ai`, then call `POST /api/v1/tenant/preview-domain/verify`. MIOSA auto-provisions a wildcard TLS certificate once the CNAME resolves. Only apex domains are accepted  -  subdomains and protocol prefixes are rejected.

One preview domain per tenant. The domain applies to all sandboxes under that tenant.

```bash
# Set
curl -X PUT https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"preview_domain": "yourproduct.com"}'

# Verify
curl -X POST https://api.miosa.ai/api/v1/tenant/preview-domain/verify \
  -H "Authorization: Bearer msk_u_..."
```

[Tenant Preview Domain API →](/docs/api-reference/tenant-preview-domain)

## Tenant branding

**Tenant branding** controls what your end users see on MIOSA-rendered error pages  -  the 404, 502, and gateway-timeout pages displayed when a sandbox is unavailable, a preview URL is invalid, or a custom domain is not yet verified.

Configurable fields:

| Field | Description |
|-------|-------------|
| `logo_url` | HTTPS URL to your logo image |
| `name` | Product or company name |
| `support_url` | Optional link shown on error pages for end-user support |
| `primary_color` | Hex color for CTA buttons (for example `#0066FF`) |

Branding is set via `PUT /api/v1/tenant/branding` and served immediately from cache  -  there is no propagation delay. Deleting it resets the error pages to the MIOSA defaults.

```bash
# Set branding
curl -X PUT https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "branding": {
      "logo_url": "https://cdn.example.com/logo.png",
      "name": "Lumen",
      "primary_color": "#0066FF"
    }
  }'
```

[Tenant Branding API →](/docs/api-reference/tenant-branding)

## Next

- [Preview and share tokens](/docs/concepts/tokens): browser access to a sandbox preview.
- [Tenant Preview Domain API](/docs/api-reference/tenant-preview-domain): the full request schema.
- [Core Concepts](/docs/concepts): the one-screen overview.
