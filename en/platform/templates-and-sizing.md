---
slug: "platform/templates-and-sizing"
title: "Templates and sizing"
description: "Pick a template and a size for each user's machine, cap concurrency, and set idle defaults so unused machines pause."
section: "Platform builders"
order: 9004
---

Every machine you make for a user needs a template and a size.
This page covers the defaults a platform builder sets, and where the full resource contracts live.

## Templates

A platform typically starts every user's machine from one template.

**TypeScript**

```typescript
const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  workspaceId: workspace.id,
  externalUserId: user.id,
})
```

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    workspace_id=workspace.id,
    external_user_id=user.id,
)
```

## Sizes and idle defaults

Sizes, named resource contracts, and idle auto-pause defaults are documented once, for every sandbox:

- [Sizing and limits](/docs/develop/sandboxes/sizing): size names, resource ranges, and the idle auto-pause default.
- [Timeouts](/docs/develop/sandboxes/timeouts): `timeout_sec`, `always_on`, and `idle_timeout_sec`.
- [Sandbox lifecycle](/docs/develop/sandboxes/lifecycle): pause, resume, and persistence.

Cap concurrency and runtime, and set idle timeouts so unused machines pause.

## Quotas and caps

One machine per end user.
Set per-user limits so one account cannot exhaust the organization's capacity: [Quotas](/docs/api-reference/quotas).

## Costs and billing

Every usage event carries the workspace, project, and your `external_*` IDs.
Group by whichever one you charge on.

```typescript
const report = await miosa.usage.report({
  period: { start: "2026-05-01", end: "2026-06-01" },
  groupBy: ["external_workspace_id"],
})

for (const row of report.data) {
  await charge(row.external_workspace_id, row.total_usd)
}
```

MIOSA sends the organization one invoice for everything.
It does not bill your customers.
You decide the markup and charge them on your own system.
Subscribe to signed lifecycle events with `miosa.webhooks.create`, and verify the `x-miosa-signature` HMAC (SHA-256 of the raw body) before acting on a delivery.
See [Usage and billing](/docs/platform/usage-and-billing) and [Webhooks](/docs/integrations/webhooks).

Persist `workspace_id` and `project_id` beside your own IDs on first create.
Query usage every period and enforce your own spend caps.

## Next

- [Sizing and limits](/docs/develop/sandboxes/sizing): Named resource contracts and the idle auto-pause default.
- [Quotas](/docs/api-reference/quotas): Per-user limits so one account cannot exhaust the organization.
- [Usage and billing](/docs/platform/usage-and-billing): Per-customer usage and chargeback on a single MIOSA bill.
