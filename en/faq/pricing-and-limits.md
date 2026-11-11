---
slug: "faq/pricing-and-limits"
title: "Pricing and limits"
description: "What you are billed for, paused cost, named sizes, hard caps, rate limits, and credits."
section: "Help"
order: 16009
---

Short answers on what is billable, the sizes and caps, and how credits work.

## What am I billed for?

Running sandboxes and computers at the hourly price of their size, apps and databases on what they use (never more than the price of their size), paused storage, storage and egress, and builds.
The full list is on [Pricing and limits](/docs/pricing).

```bash
miosa usage my-box           # runtime, time left and estimated cost
```

## Do paused sandboxes cost anything?

Storage only. Billing for compute stops when the sandbox leaves `running`.
See [Sizing and limits](/docs/develop/sandboxes/sizing).

## What are the named sizes?

`xs` (1 vCPU, 2 GiB, 10 GiB) through `xl` (16 vCPU, 32 GiB, 80 GiB), with `small` as the default.
See [Sizing and limits](/docs/develop/sandboxes/sizing).

## What are the hard caps?

Concurrent sandboxes per plan (Developer 10, Business 250, Enterprise 500; upgrade or raise it through support), `timeout_sec` 1 to 86400, an 86,400 second maximum exec timeout, a 500 MB maximum upload, and a 1 hour browser token TTL.
See [Pricing and limits](/docs/pricing).

## What are the rate limits?

60 to 30,000 requests per minute per API key by tier, and 2,500/25,000/100,000 per minute per workspace on Developer/Business/Enterprise, with `429` and `Retry-After` on overrun.
See [Rate limits](/docs/rate-limits).

## How do I cap what one end user can spend?

The [Quotas API](/docs/api-reference/quotas) sets `max_sandboxes`, `max_concurrent`, `max_storage_gb`, and `max_credit_cents` per `external_user_id`, enforced at create time.

## How do I charge my own customers?

MIOSA bills you once; you charge your customers using attribution.
The [Usage Rollup API](/docs/api-reference/usage-rollup) groups by `external_user_id`, `external_project_id`, or `workspace_id`.
See [Usage and billing](/docs/platform/usage-and-billing).

## Do credits expire?

It depends on the grant.
Credits you add do not expire.
Recurring plan credits expire at the start of the UTC day after the 30-day billing period.
Promotional credits (the one-time welcome grant and promo-code redemptions) expire 180 days after they are granted.
The nearest expiry is reported as `credit_expiry_at` on `GET /api/v1/credits/balance`.
See [Pricing and limits](/docs/pricing).
