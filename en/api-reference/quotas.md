---
slug: "api-reference/quotas"
title: "Quotas API"
description: "Set per-end-user resource limits for white-label multi-tenant deployments."
section: "API reference"
order: 12047
---

Quotas let you cap how many sandboxes an individual end-user of your product can create and run. They are keyed by `external_user_id`  -  the same opaque identifier you pass when creating sandboxes on behalf of your users.

Base path: `/api/v1/quotas/external/{external_user_id}`

> **Note**
>
> Quotas are enforced at sandbox creation time. Setting a quota does not affect sandboxes already running. Deleting a quota removes all limits for that user.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/quotas/external/{external_user_id}` | Get quota and current usage |
| `PUT` | `/api/v1/quotas/external/{external_user_id}` | Create or update a quota |
| `DELETE` | `/api/v1/quotas/external/{external_user_id}` | Remove quota (unlimited) |

---

## Get Quota

**`GET /api/v1/quotas/external/{external_user_id}`**

Returns the configured limits and live usage counters for the user.

### Auth

```http
Authorization: Bearer msk_...
```

### Response  -  `200 OK`

```json
{
  "data": {
    "external_user_id": "user_42",
    "limits": {
      "max_sandboxes": 10,
      "max_concurrent": 3,
      "max_storage_gb": 50,
      "max_credit_cents": 5000
    },
    "usage": {
      "total_sandboxes": 4,
      "concurrent": 1
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `limits.max_sandboxes` | integer\|null | Lifetime sandbox creation limit |
| `limits.max_concurrent` | integer\|null | Max sandboxes running simultaneously |
| `limits.max_storage_gb` | integer\|null | Total attached storage cap in GiB |
| `limits.max_credit_cents` | integer\|null | Spend cap in cents (platform credits) |
| `usage.total_sandboxes` | integer | Total sandboxes created by this user |
| `usage.concurrent` | integer | Currently running sandboxes |

A `null` limit means no cap is applied for that dimension.

---

## Set Quota

**`PUT /api/v1/quotas/external/{external_user_id}`**

Creates or replaces the quota. All limit fields are optional  -  omit a field to leave it uncapped.

### Request Body

```json
{
  "max_sandboxes": 10,
  "max_concurrent": 3,
  "max_storage_gb": 50,
  "max_credit_cents": 5000
}
```

### Response  -  `200 OK`

Same shape as GET response.

---

## Delete Quota

**`DELETE /api/v1/quotas/external/{external_user_id}`**

Removes all limits for the user. Future sandbox creation is governed only by your tenant plan.

### Response  -  `204 No Content`

---

## Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 | `not_found` | No quota exists for this `external_user_id` (GET/DELETE) |
| 422 | validation errors | A limit value is non-integer or negative |

---

## Examples

**cURL**

```bash
# Set
curl -X PUT https://api.miosa.ai/api/v1/quotas/external/user_42 \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"max_sandboxes": 10, "max_concurrent": 3}'

# Get
curl https://api.miosa.ai/api/v1/quotas/external/user_42 \
  -H "Authorization: Bearer msk_u_..."

# Remove
curl -X DELETE https://api.miosa.ai/api/v1/quotas/external/user_42 \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Set quota when user signs up
client.quotas.set("user_42", max_sandboxes=10, max_concurrent=3)

# Check quota + usage
quota = client.quotas.get("user_42")
print(quota.usage.concurrent)

# Remove limits
client.quotas.delete("user_42")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Set
await client.quotas.set('user_42', {
  maxSandboxes: 10,
  maxConcurrent: 3,
  maxStorageGb: 50,
});

// Get
const quota = await client.quotas.get('user_42');
console.log(quota.usage.concurrent);

// Remove
await client.quotas.delete('user_42');
```
