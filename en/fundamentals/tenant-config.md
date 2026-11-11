---
slug: "fundamentals/tenant-config"
title: "Tenant configuration"
description: "Tenant-level API surfaces for white-label integrators - webhook subscriptions and per-end-user quotas."
section: "API reference"
order: 12006
---

White-label integrators configure tenant behavior through the API surfaces below. All routes accept `msk_a_*` admin keys or admin JWTs.

Preview domains and branding are part of the product model, not this integrator guide: [Core Concepts](/docs/concepts#tenant-preview-domain) explains what they are and when to use them, and the [Tenant Preview Domain](/docs/api-reference/tenant-preview-domain) and [Tenant Branding](/docs/api-reference/tenant-branding) references give the request schemas.

## Webhooks

Subscribe to sandbox and deployment lifecycle events delivered as HTTPS POST requests to your endpoint. MIOSA signs each request with an HMAC-SHA256 signature in the `X-Miosa-Signature` header:

```bash
POST /api/v1/tenant/webhooks
{ "url": "https://yourapp.com/webhooks/miosa", "events": ["sandbox.*", "deployment.*"] }
```

See [Webhooks](/docs/api-reference/webhooks) and [Integrations](/docs/integrations) for the event catalog and signature verification.

## Per-end-user quotas

Set maximum resource counts per `external_user_id` to prevent runaway usage by a single downstream user. Quotas are checked at resource creation time and return `429` with code `QUOTA_EXCEEDED` when reached:

```bash
PUT /api/v1/tenant/quotas
{ "max_sandboxes_per_user": 5, "max_computers_per_user": 2 }
```

See [Quotas](/docs/api-reference/quotas).
