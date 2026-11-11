---
slug: "platform/usage-and-billing"
title: "Usage & Billing"
description: "Track MIOSA usage per workspace, per project, per external customer, and per user. Chargeback to your customers."
section: "Organization & billing"
order: 13005
---

If you're a platform reselling MIOSA capacity, you need to attribute MIOSA costs back to your own customers. MIOSA emits usage events with [resource ownership and external attribution](/docs/platform/attribution) on every billable action; you query them, group, and chargeback in your own currency / margin model.

> **Note: Current status**
>
> Tenant-scoped usage rollup, session usage, summary, workspace usage, and audit-log endpoints are live.
> The exact dimensions available in a response depend on the resource and metering source.
> Treat missing measurements as unavailable, not zero.

## What's metered

| Dimension | Unit | When emitted |
|---|---|---|
| Sandbox runtime | sandbox-seconds, priced by size | Per-second while running |
| Builder runtime | builder-seconds | During publish |
| Runtime instance runtime | runtime-seconds, measured CPU and memory, capped at the size price | Per-second while serving |
| Static artifact storage | GB-months | Continuously |
| Static artifact bandwidth | GB egress | Per response |
| Object storage (buckets) | GB-months + GB egress | Continuously / per response |
| Managed Postgres | GB-months + connections | Continuously |
| Managed Redis | GB-months | Continuously |
| Ownership and attribution: every event carries `workspace_id`, `project_id`, and the `external_*` fields if present on the source resource |

Computers (desktop product) have a separate metering schema; see [Computers](/docs/computers/overview) for that side.

## Usage report API
```http
GET /api/v1/usage?period_start=2026-05-01&period_end=2026-06-01
GET /api/v1/usage?workspace_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/usage?external_workspace_id=acct_123&period=2026-05
```

**TypeScript**

```typescript
const report = await miosa.usage.report({
  period: { start: "2026-05-01", end: "2026-06-01" },
  groupBy: ["external_workspace_id"],
})

for (const row of report.data) {
  console.log(
    row.external_workspace_id,
    "$" + row.total_usd.toFixed(2),
    row.line_items
  )
}
```

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Grouped chargeback report for a billing period
report = miosa.usage.report(
    period_start="2026-05-01",
    period_end="2026-06-01",
    group_by=["external_workspace_id"],
)

for row in report.get("data", []):
    print(row["external_workspace_id"], f"${row['total_usd']:.2f}")

# Single-workspace query for the current period
summary = miosa.usage.report(
    external_workspace_id="acct_123",
)
print(summary)

# Raw per-session events
sessions = miosa.usage.sessions(external_workspace_id="acct_123")
for s in sessions:
    print(s["dimension"], s["qty"])
```

Response shape:
```json
{
  "data": [
    {
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "project_id": "660e8400-e29b-41d4-a716-446655440001",
      "external_workspace_id": "acct_123",
      "external_user_id": null,
      "external_project_id": null,
      "total_usd": 1.89,
      "line_items": [
        { "dimension": "sandbox_seconds", "qty": 36000, "usd": 0.36 },
        { "dimension": "static_bandwidth_gb", "qty": 38.2, "usd": 1.53 }
      ]
    },
    {
      "external_workspace_id": "acct_456",
      "total_usd": 11.75,
      "line_items": [...]
    }
  ],
  "period": { "start": "2026-05-01", "end": "2026-06-01" },
  "currency": "usd"
}
```

Use `groupBy` to slice by any combination of ownership and external IDs. Typical groupings:

- `["workspace_id"]`  -  per-client workspace
- `["workspace_id", "project_id"]`  -  per-app/project inside a client
- `["external_workspace_id"]`  -  per-customer chargeback
- `["external_workspace_id", "external_project_id"]`  -  per-app within a customer
- `["external_user_id"]`  -  per-user attribution (e.g. for fair-use limits)
- `[]`  -  total for the entire MIOSA organization

## Audit log

Every mutation also emits an audit event with attribution. Different table, separate read endpoint:
```http
GET /api/v1/audit-log?workspace_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/audit-log?external_workspace_id=acct_123
GET /api/v1/audit-log?api_key_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/audit-log?action=deployments.publish
```

Audit is for security / compliance, usage is for billing. Both share the same ownership and attribution model.

## Quotas and limits

MIOSA enforces quotas at the organization and plan level. You can enforce per-workspace, per-project, or per-user limits in your own app logic, querying usage to make decisions.

Example: Lumen wants to cap each customer account at $50/month. Their backend:

1. Queries `/api/v1/usage?workspace_id=550e8400-e29b-41d4-a716-446655440000&period=current_month`.
2. If `total_usd &gt; 50`, denies new sandbox creation in the Lumen app.
3. Optionally, hits a MIOSA endpoint to suspend the customer's sandboxes (not a single endpoint today; do it client-side by destroying their sandboxes).

MIOSA also exposes external-user quota endpoints at `/api/v1/quotas/external/{external_user_id}`.
Use them when the caller should be prevented from creating more resources after reaching a configured limit.

## Pricing

Every size has one price, the same on every plan; your plan sets limits, never the price. See [Pricing](/docs/pricing) for the rates, or `GET /api/v1/pricing` for the live price list.

## Billing tenancy

Important: your MIOSA bill is one bill. MIOSA charges the organization owner for all usage, regardless of which workspace/project or `external_workspace_id` it came from. Ownership and attribution exist so you can charge your customers. MIOSA does not bill your downstream customers directly.
```text

MIOSA -> Lumen:        $X / month, one invoice
Lumen -> acct_123:   $Y / month (markup, your pricing model)
Lumen -> acct_456:   $Z / month
```

## See also

- [Ownership and Attribution](/docs/platform/attribution)  -  the IDs that flow through usage events
- [API Keys](/docs/platform/api-keys)  -  the credential for usage queries (`usage:read` scope)
- [Audit](/docs/api-reference/events)  -  for security log queries
