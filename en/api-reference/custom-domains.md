---
slug: "api-reference/custom-domains"
title: "Custom Domains API"
description: "API reference for mapping tenant-owned FQDNs to MIOSA previews with automatic TLS."
section: "API reference"
order: 12043
---

Custom domains let you expose MIOSA previews, computers, and deployments under tenant-owned FQDNs. There are four related surfaces:

- **Computer custom domains** map one fully qualified domain, such as `app.yourdomain.com`, to one Computer service.
- **Deployment custom domains** map one fully qualified domain, such as `program.example.net`, to one Deployment.
- **Tenant preview domains** white-label generated preview links for the whole organization. After configuration, sandbox `/expose` responses use URLs such as `https://5173-sbx01j9x.sandbox.example.com`.

MIOSA handles verification and automatic TLS certificate issuance via Caddy on-demand TLS.

Nested base paths: `/api/v1/computers/{id}/domains` and `/api/v1/deployments/{id}/domains`.

Flat frontend-compatible base path: `/api/v1/custom-domains`.

> **Note**
>
> Custom domains must not end in `miosa.ai`  -  that namespace is reserved for platform subdomains. Use a domain you control.

---

## Quick Start
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// 1. Register the domain
const domain = await client.domains.register(computerId, {
  domain: 'app.yourcompany.com',
});
console.log(domain.verificationTarget); // e.g. "verify.miosa.ai"

// 2. Add a CNAME record at your DNS provider:
//    app.yourcompany.com → verify.miosa.ai

// 3. Trigger verification
const verified = await client.domains.verify(computerId, domain.id);
console.log(verified.status); // "verified" or "failed"
```
```bash
# Register
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain": "app.yourcompany.com"}'
```

---

## Endpoints

### Computer custom domains

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/computers/{id}/domains` | Register a custom domain |
| `GET` | `/computers/{id}/domains` | List domains for a computer |
| `POST` | `/computers/{id}/domains/{domain_id}/verify` | Trigger DNS verification |
| `DELETE` | `/computers/{id}/domains/{domain_id}` | Remove a domain |

### Deployment custom domains

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/deployments/{id}/domains` | Register a custom domain |
| `GET` | `/deployments/{id}/domains` | List domains for a deployment |
| `POST` | `/deployments/{id}/domains/{domain_id}/verify` | Trigger DNS verification |
| `DELETE` | `/deployments/{id}/domains/{domain_id}` | Remove a domain |

### Flat custom domains

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/custom-domains?computer_id={id}` | List domains for a computer |
| `GET` | `/custom-domains?deployment_id={id}` | List domains for a deployment |
| `POST` | `/custom-domains` | Register a domain with `computer_id` or `deployment_id` |
| `DELETE` | `/custom-domains/{id}` | Remove a domain |

### Tenant preview domains

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/tenant/preview-domain` | Read the tenant preview domain |
| `PUT` | `/tenant/preview-domain` | Set the tenant preview domain |
| `DELETE` | `/tenant/preview-domain` | Clear the tenant preview domain |
| `POST` | `/tenant/preview-domain/verify` | Check DNS/TLS readiness |

The preview domain is an organization setting. There is no workspace-level or project-level preview-domain endpoint.

---

## White-label Managed Links

Use a tenant preview domain when generated app/artifact preview URLs should use your organization's domain instead of the MIOSA-managed fallback.
```bash
curl -X PUT https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"preview_domain":"preview.yourcompany.com"}'
```

Then expose a sandbox port:
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/expose \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"port":5173}'
```

Response:
```json
{
  "url": "https://5173-sbx01j9x.sandbox.preview.yourcompany.com"
}
```

For a single deployment under its own FQDN (not the shared preview domain), register a deployment custom domain instead:

```bash
curl -X POST https://api.miosa.ai/api/v1/deployments/$DEPLOYMENT_ID/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain":"program.example.net"}'
```

DNS must delegate wildcard traffic to the MIOSA preview router:

| Record type | Name | Value | Used for |
|---|---|---|---|
| `CNAME` | `*` | `proxy.miosa.ai` | `https://{slug}.{domain}` managed deployment/computer/default preview URLs, and `https://{port}-{slug}.sandbox.{domain}` sandbox port previews |

If no tenant preview domain is configured, MIOSA returns the managed fallback URL for that resource. This fallback remains available even after custom domains are attached.

Exact deployment and computer custom domains always win over the organization-wide preview domain.

---

## Register a Domain

**`POST /api/v1/computers/{id}/domains`**

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `domain` | string | Yes | Fully qualified domain name. RFC 1123 hostname, max 253 chars |

### Response  -  `201 Created`
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "computer_id": "...",
    "deployment_id": null,
    "tenant_id": "...",
    "workspace_id": "660e8400-e29b-41d4-a716-446655440001",
    "project_id": "770e8400-e29b-41d4-a716-446655440002",
    "fqdn": "app.yourcompany.com",
    "status": "pending",
    "verification_target": "verify.miosa.ai",
    "verified_at": null,
    "tls_issued_at": null,
    "external_workspace_id": "acct_123",
    "external_user_id": "user-456",
    "external_project_id": "project_789",
    "created_at": "2026-04-11T00:00:00Z",
    "updated_at": "2026-04-11T00:00:00Z"
  }
}
```

### Status Values

| Status | Description |
|--------|-------------|
| `pending` | Registered; awaiting DNS verification |
| `verified` | CNAME confirmed; Caddy may issue a cert |
| `active` | TLS certificate issued and in use |
| `failed` | Verification or TLS issuance failed |
| `removed` | Domain detached (soft marker before deletion) |

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 409 | `has already been taken` | FQDN already registered (globally unique) |
| 422 | Validation error | Invalid FQDN format or ends with `miosa.ai` |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain": "preview.yourapp.io"}'
```

### Register a deployment domain
```bash
curl -X POST https://api.miosa.ai/api/v1/deployments/{id}/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"domain": "program.example.net"}'
```

Deployment domains inherit `workspace_id`, `project_id`, and external attribution from the deployment.

---

## List Domains

**`GET /api/v1/computers/{id}/domains`**

### Response  -  `200 OK`
```json
{
  "data": [
    {
      "id": "...",
      "fqdn": "app.yourcompany.com",
      "status": "active",
      "verified_at": "2026-04-11T01:00:00Z",
      "tls_issued_at": "2026-04-11T01:05:00Z",
      "created_at": "2026-04-11T00:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/domains \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Verify a Domain

**`POST /api/v1/computers/{id}/domains/{domain_id}/verify`**

Checks that the CNAME record at `fqdn` resolves to `verification_target`. On success the domain transitions to `"verified"` and Caddy will issue a TLS certificate on the next HTTPS request to the FQDN.

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | UUID | Computer ID |
| `domain_id` | UUID | Domain ID |

### Response  -  `200 OK`
```json
{
  "data": {
    "id": "...",
    "fqdn": "app.yourcompany.com",
    "status": "verified",
    "verified_at": "2026-04-11T01:00:00Z"
  }
}
```

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 409 | `domain is already verified` | Already completed |
| 422 | `CNAME not found` | DNS record missing or not yet propagated |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/domains/{domain_id}/verify \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Remove a Domain

**`DELETE /api/v1/computers/{id}/domains/{domain_id}`**

Detaches the domain from the computer. The certificate is not revoked immediately (Caddy lets it expire naturally), but the domain will no longer route to the computer.

### Response  -  `200 OK`
```json
{
  "data": { "id": "...", "fqdn": "app.yourcompany.com", "status": "removed" }
}
```
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/domains/{domain_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## DNS Setup Reference

After registering a domain, add the following DNS record at your registrar or DNS provider:

| Record | Host | Value |
|--------|------|-------|
| CNAME | `app` (or full subdomain) | `verify.miosa.ai` |

DNS propagation typically takes 1-60 minutes. Call `verify` once propagation is complete. If verification fails, wait a few minutes and retry  -  the endpoint is idempotent.

---

## Common Recipes

### Automate full domain onboarding
```python
import time
from miosa import Miosa

client = Miosa()

domain = client.domains.register(computer_id, fqdn="api.yourcompany.com")
print(f"Add CNAME: {domain.fqdn} → {domain.verification_target}")
print("Waiting for DNS propagation...")

while True:
    result = client.domains.verify(computer_id, domain.id)
    if result.status == "verified":
        print("Domain verified! TLS will be issued on first HTTPS request.")
        break
    elif result.status == "failed":
        raise RuntimeError("Verification failed  -  check your DNS records.")
    time.sleep(30)
```

### List all pending domains (potential stuck ACME issuances)
```typescript
const computers = await client.computers.list();
for (const computer of computers.data) {
  const { data: domains } = await client.domains.list(computer.id);
  const pending = domains.filter(d => d.status === 'pending');
  if (pending.length > 0) {
    console.log(`${computer.name}: ${pending.length} domain(s) pending verification`);
  }
}
```
