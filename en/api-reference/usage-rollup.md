---
slug: "api-reference/usage-rollup"
title: "Usage Rollup API"
description: "Aggregate sandbox compute usage and spend by external user, project, or workspace."
section: "API reference"
order: 12048
---

The usage rollup endpoint aggregates metered usage across sandboxes and computers, grouped by an identifier of your choice. Use it to build per-user billing dashboards, enforce spend policies, or attribute costs to projects.

Base path: `/api/v1/usage`

> **Note**
>
> Results are limited to 1000 rows per request. For high-cardinality tenants, narrow the period or use a more specific `group_by`.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/usage` | Aggregate usage for a period |

---

## Get Usage Rollup

**`GET /api/v1/usage`**

### Auth

```http
Authorization: Bearer msk_...
```

### Query Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `group_by` | string | No | `external_user_id` | Dimension to aggregate by |
| `period` | string | No | `30d` | Predefined period or `custom` |
| `start` | string | No |  -  | ISO 8601 start (required when `period=custom`) |
| `end` | string | No |  -  | ISO 8601 end (required when `period=custom`) |

**`group_by` values:**

| Value | Description |
|-------|-------------|
| `external_user_id` | Group by the `external_user_id` you set on sandboxes |
| `external_project_id` | Group by `external_project_id` |
| `workspace_id` | Group by MIOSA workspace |

**`period` values:**

| Value | Description |
|-------|-------------|
| `7d` | Last 7 days |
| `30d` | Last 30 days |
| `mtd` | Month-to-date (calendar month, resets on the 1st) |
| `custom` | Arbitrary range  -  requires `start` and `end` |

### Response  -  `200 OK`

```json
{
  "period_start": "2026-04-26T00:00:00Z",
  "period_end": "2026-05-26T00:00:00Z",
  "results": [
    {
      "external_user_id": "user_42",
      "sandbox_seconds": 14400,
      "computer_seconds": 0,
      "storage_gb_hours": 2.5,
      "credit_cents": 312
    },
    {
      "external_user_id": "user_99",
      "sandbox_seconds": 3600,
      "computer_seconds": 7200,
      "storage_gb_hours": 0.0,
      "credit_cents": 108
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `period_start` | string | ISO 8601 start of aggregation window |
| `period_end` | string | ISO 8601 end of aggregation window |
| `results[].{group_by}` | string | Value of the grouping dimension |
| `results[].sandbox_seconds` | number | vCPU-seconds used by sandboxes |
| `results[].computer_seconds` | number | vCPU-seconds used by computers |
| `results[].storage_gb_hours` | number | GiB-hours of attached storage |
| `results[].credit_cents` | number | Total cost in platform credit cents |

Results are ordered by `credit_cents` descending.

---

## Errors

| Status | Code | Cause |
|--------|------|-------|
| 422 | `invalid group_by` | `group_by` is not one of the accepted values |
| 422 | `invalid period` | `period` is not one of the accepted values |
| 422 | `invalid_custom_range` | `start` or `end` is missing or not valid ISO 8601 when `period=custom` |

---

## Examples

**cURL**

```bash
# 30-day rollup by user
curl "https://api.miosa.ai/api/v1/usage?group_by=external_user_id&period=30d" \
  -H "Authorization: Bearer msk_u_..."

# Custom range
curl "https://api.miosa.ai/api/v1/usage?group_by=external_user_id&period=custom&start=2026-05-01T00:00:00Z&end=2026-05-26T00:00:00Z" \
  -H "Authorization: Bearer msk_u_..."

# Group by project
curl "https://api.miosa.ai/api/v1/usage?group_by=external_project_id&period=mtd" \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Monthly rollup per user
results = client.usage.rollup(
    group_by="external_user_id",
    period="30d",
)

for row in results:
    print(f"{row.external_user_id}: {row.credit_cents / 100:.2f} USD")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

const { results } = await client.usage.rollup({
  groupBy: 'external_user_id',
  period: '30d',
});

for (const row of results) {
  console.log(`${row.externalUserId}: $${(row.creditCents / 100).toFixed(2)}`);
}
```
