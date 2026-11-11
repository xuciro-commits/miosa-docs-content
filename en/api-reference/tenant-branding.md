---
slug: "api-reference/tenant-branding"
title: "Tenant Branding API"
description: "Customize the logo, name, and colors shown on sandbox 404 and 502 error pages."
section: "API reference"
order: 12051
---

The branding API controls the appearance of proxy-generated error pages (404, 502, gateway timeout) served within your custom preview domain. Set your logo, product name, support URL, and primary accent color so end-users see your brand  -  not MIOSA's.

Base path: `/api/v1/tenant/branding`

> **Note**
>
> Branding changes are served immediately from cache. There is no propagation delay.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/tenant/branding` | Get current branding |
| `PUT` | `/api/v1/tenant/branding` | Set or replace branding |
| `DELETE` | `/api/v1/tenant/branding` | Reset to MIOSA defaults |

---

## Get Branding

**`GET /api/v1/tenant/branding`**

### Auth

```http
Authorization: Bearer msk_...
```

### Response  -  `200 OK`

```json
{
  "branding": {
    "logo_url": "https://cdn.example.com/logo.png",
    "name": "Lumen",
    "support_url": "https://example.com/support",
    "primary_color": "#0066FF"
  }
}
```

Returns `{"branding": null}` if no custom branding has been set.

---

## Set Branding

**`PUT /api/v1/tenant/branding`**

Replaces all branding fields. Omitted fields are cleared to their defaults.

### Request Body

Wrap fields under a `"branding"` key.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `logo_url` | string | No | HTTPS URL to your logo image |
| `name` | string | No | Product or company name |
| `support_url` | string | No | Link shown on error pages for user support |
| `primary_color` | string | No | Hex color for CTA buttons (e.g. `"#0066FF"`) |

```json
{
  "branding": {
    "logo_url": "https://cdn.example.com/logo.png",
    "name": "Lumen",
    "support_url": "https://example.com/support",
    "primary_color": "#0066FF"
  }
}
```

### Response  -  `200 OK`

```json
{
  "branding": {
    "logo_url": "https://cdn.example.com/logo.png",
    "name": "Lumen",
    "support_url": "https://example.com/support",
    "primary_color": "#0066FF"
  }
}
```

---

## Delete Branding

**`DELETE /api/v1/tenant/branding`**

Resets all branding to MIOSA defaults. Error pages will show the MIOSA logo and color scheme.

### Response  -  `200 OK`

```json
{
  "branding": null
}
```

---

## Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 | `tenant not found` | Tenant does not exist |
| 422 | validation error | A field failed validation (e.g. invalid URL, invalid hex color) |

---

## Examples

**cURL**

```bash
# Set branding
curl -X PUT https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "branding": {
      "logo_url": "https://cdn.example.com/logo.png",
      "name": "Lumen",
      "support_url": "https://example.com/support",
      "primary_color": "#0066FF"
    }
  }'

# Get
curl https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_u_..."

# Reset
curl -X DELETE https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

client.tenant.set_branding(
    logo_url="https://cdn.example.com/logo.png",
    name="Lumen",
    support_url="https://example.com/support",
    primary_color="#0066FF",
)

branding = client.tenant.get_branding()
print(branding.name)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

await client.tenant.setBranding({
  logoUrl: 'https://cdn.example.com/logo.png',
  name: 'Lumen',
  supportUrl: 'https://example.com/support',
  primaryColor: '#0066FF',
});

const branding = await client.tenant.getBranding();
console.log(branding?.name);
```
