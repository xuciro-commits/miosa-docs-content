---
slug: "api-reference/tenant-preview-domain"
title: "Tenant Preview Domain API"
description: "Configure a custom apex domain for sandbox preview URLs (white-label)."
section: "API reference"
order: 12052
---

The preview domain API lets you map your own apex domain (e.g. `example.com`) to sandbox preview URLs. When configured, sandboxes created under your tenant will be accessible at `{sandbox-slug}.{your-domain}` instead of the default `*.sandbox.miosa.ai`.

> **Note**
>
> **Auth note:** These endpoints accept both `msk_*` API keys and JWT tokens. This matters for white-label integrations where you programmatically manage the domain from a backend service.

> **Note**
>
> Only apex domains are accepted (e.g. `example.com`, not `app.example.com` or `https://example.com`). Wildcard and subdomain inputs are rejected with `422 invalid_domain`.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/tenant/preview-domain` | Get current domain and verification status |
| `PUT` | `/api/v1/tenant/preview-domain` | Set or replace the preview domain |
| `POST` | `/api/v1/tenant/preview-domain/verify` | Recheck DNS verification |
| `DELETE` | `/api/v1/tenant/preview-domain` | Remove the custom domain |

---

## Get Preview Domain

**`GET /api/v1/tenant/preview-domain`**

Returns the configured domain and its DNS verification status.

### Auth

```http
Authorization: Bearer msk_...
```

### Response  -  `200 OK` (domain configured)

```json
{
  "preview_domain": "example.com",
  "status": "configured",
  "dns_instructions": {
    "type": "CNAME",
    "name": "*.example.com",
    "value": "proxy.miosa.ai"
  }
}
```

### Response  -  `200 OK` (not configured)

```json
{
  "preview_domain": null,
  "default_domain": "miosa.ai",
  "status": "not_configured",
  "dns_instructions": null
}
```

| Field | Type | Description |
|-------|------|-------------|
| `preview_domain` | string\|null | Configured apex domain, or `null` |
| `default_domain` | string | Fallback domain used when no custom domain is set |
| `status` | string | `"not_configured"`, `"pending_dns"`, or `"configured"` |
| `dns_instructions` | object\|null | CNAME record to add at your DNS registrar |

---

## Set Preview Domain

**`PUT /api/v1/tenant/preview-domain`**

Sets or replaces the custom preview domain. DNS verification is asynchronous  -  status transitions from `pending_dns` to `configured` once the CNAME resolves correctly.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `preview_domain` | string | Yes | Apex domain (e.g. `"example.com"`) |

```json
{
  "preview_domain": "example.com"
}
```

### Response  -  `200 OK`

```json
{
  "preview_domain": "example.com",
  "status": "pending_dns",
  "dns_status": "pending",
  "dns_instructions": {
    "type": "CNAME",
    "name": "*.example.com",
    "value": "proxy.miosa.ai"
  }
}
```

### DNS Setup

Add a wildcard CNAME at your registrar:

| Type | Name | Value |
|------|------|-------|
| `CNAME` | `*` (or `*.example.com`) | `proxy.miosa.ai` |

---

## Verify DNS

**`POST /api/v1/tenant/preview-domain/verify`**

Triggers an immediate DNS re-check. Returns the current verification status without waiting for the background poller.

### Response  -  `200 OK`

```json
{
  "preview_domain": "example.com",
  "status": "configured",
  "dns_status": "verified"
}
```

---

## Delete Preview Domain

**`DELETE /api/v1/tenant/preview-domain`**

Removes the custom domain. Sandbox preview URLs revert to `*.sandbox.miosa.ai`.

### Response  -  `200 OK`

```json
{
  "preview_domain": null,
  "status": "removed"
}
```

---

## Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `preview_domain is required` | Missing body field |
| 422 | `invalid_domain` | Protocol prefix, wildcard, path, or subdomain detected |
| 404 | `tenant not found` | Tenant does not exist |

---

## Examples

**cURL**

```bash
# Set
curl -X PUT https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"preview_domain": "example.com"}'

# Verify
curl -X POST https://api.miosa.ai/api/v1/tenant/preview-domain/verify \
  -H "Authorization: Bearer msk_u_..."

# Get
curl https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_u_..."

# Delete
curl -X DELETE https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Set
result = client.tenant.set_preview_domain("example.com")
print(result.dns_instructions)

# Poll until verified
import time
while True:
    domain = client.tenant.get_preview_domain()
    if domain.status == "configured":
        break
    time.sleep(30)
    client.tenant.verify_preview_domain()
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Set
const result = await client.tenant.setPreviewDomain('example.com');
console.log(result.dnsInstructions);

// Get
const domain = await client.tenant.getPreviewDomain();
console.log(domain.status); // "pending_dns" | "configured"

// Force re-check
await client.tenant.verifyPreviewDomain();

// Remove
await client.tenant.deletePreviewDomain();
```
