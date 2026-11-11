---
slug: "plans-and-limits"
title: "Plans and limits"
description: "The four MIOSA plans, what each includes, and the limits per service and per account."
section: "Organization & billing"
order: 13004
---

There are four plans.
Every plan pays the same published price for a size.
Plans differ in what they include and in how much you can run.

| Plan | Price | Included usage each month | Past the included usage |
|---|---|---|---|
| Free | $0 | $1 | Stops at $1. Add credits to keep going. |
| Developer | $20/mo | $20 | Pay as you go. Add credits any time. |
| Business | $500/mo | $500 | Pay as you go. Add credits any time. |
| Enterprise | $2,000/mo | $2,000 | Pay as you go, invoiced |

Pick a plan in [Billing](https://miosa.ai/billing).
Free works for a first try.

## Per service

A service's replicas, size and storage are limited by your plan.

| Plan | Replicas | Largest replica | All replicas together | Always-on | Volume | Image |
|---|---|---|---|---|---|---|
| Free | 1 | 1 vCPU, 0.5 GB | 0.5 GB, 1 vCPU | No, it sleeps | 0.5 GB | 4 GB |
| Free with added credits | 2 | 2 vCPU, 2 GB | 2 GB, 2 vCPU | No, it sleeps | 2 GB | 8 GB |
| Developer | 4 | 4 vCPU, 8 GB | 16 GB, 8 vCPU | Yes | 10 GB | 20 GB |
| Business | 10 | 8 vCPU, 16 GB | 64 GB, 32 vCPU | Yes | 250 GB | 50 GB |
| Enterprise | 20 | 16 vCPU, 32 GB | 128 GB, 64 vCPU | Yes | 1 TB | Unlimited |

## Per account

These totals count everything running at once: services, sandboxes, computers and databases.

| Plan | Running memory | Running vCPU | Projects | Sandboxes | Computers | Databases | Egress per month | Seats |
|---|---|---|---|---|---|---|---|---|
| Free | 2 GB | 2 | 1 | 1 (stops after 30 minutes) | 1 small, hibernates after 15 idle minutes | 1 small | 5 GB | 1 |
| Free with added credits | 6 GB | 4 | 2 | 2 (stop after 2 hours) | 1, up to 2 vCPU and 4 GB | 2 | 20 GB | 1 |
| Developer | 32 GB | 16 | 10 | 5 | 3 | 5 | 100 GB | 1 |
| Business | 128 GB | 64 | Unlimited | 25 | 20 | 25 | 1 TB | 10 |
| Enterprise | 256 GB | 128 | Unlimited | 100 | 100 | 100 | 5 TB | Unlimited |

Enterprise totals can be raised by agreement.

## Free with added credits

A Free account can add credits, from $5.
Added credits never expire.
While the added balance is above $0, the "Free with added credits" limits apply.
When it reaches $0, the account is back on the Free limits.

## Dedicated vCPU

Dedicated vCPU sizes are for Business and Enterprise.
See [Dedicated vCPU](/docs/services/dedicated-vcpu).

## When you reach a limit

A create or a deploy that would pass a limit is refused with an error that names the limit and the plan that raises it.
Nothing running is stopped.
Your limits and usage are in [Billing](https://miosa.ai/billing).

## Guardrails

Two limits sit above the plan tables, and both are there to keep the fleet honest:

- **Running memory per organization.** No organization runs more than about
  100 GB of memory at once without an administrator override, whatever its plan.
- **Fleet pressure.** When the committed memory on the fleet passes 80% of what
  is free, new Free creates pause and new Free signups queue until capacity
  returns. Paid creates continue. Nothing already running is stopped.

## Where to go next

- [Pricing](/docs/pricing): the per-size rates every plan pays.
- [Credits](/docs/api-reference/credits): how credits are bought, spent and expire.
