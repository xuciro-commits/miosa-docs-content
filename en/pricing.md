---
slug: "pricing"
title: "Pricing and limits"
description: "What a sandbox costs per hour, what a plan buys, how many sandboxes you can run and start, and how that compares to other providers."
section: "Organization & billing"
order: 13003
---

What a sandbox costs per hour, what a plan buys, how many sandboxes you can run and start, and how that compares to other providers.

Every size has one price per hour. You pay it per second, and only while a sandbox or computer is running.
A paused, stopped or destroyed sandbox costs nothing for compute, and the price does not change with how busy the machine is.
Storage is metered on its own: disk costs $0.09 per GiB-month while the machine runs, and paused disk, named snapshots, and backups cost $0.018 per GiB-month. Buckets and static sites cost $0.0162 per GiB-month.
One credit is always one US cent.
Each organization keeps 10 named snapshots included; each one after that is $1.70 a month.
There are no other discounts, bonuses or free allowances: egress is billed from the first GB. Builds are free.

## Price

| Size | Per hour | Always on (730 h) | CPU | RAM | Disk |
|---|---|---|---|---|---|
| `micro` | $0.0316 | $23.05 | 1 vCPU | 0.5 GiB | 10 GiB |
| `tiny` | $0.0359 | $26.22 | 1 vCPU | 1 GiB | 20 GiB |
| `xs` | $0.0446 | $32.58 | 1 vCPU | 2 GiB | 10 GiB |
| `small` | $0.0893 | $65.16 | 2 vCPU | 4 GiB | 10 GiB |
| `quad` | $0.1437 | $104.90 | 4 vCPU | 4 GiB | 20 GiB |
| `medium` | $0.1785 | $130.32 | 4 vCPU | 8 GiB | 20 GiB |
| `large` | $0.3570 | $260.65 | 8 vCPU | 16 GiB | 40 GiB |
| `xl` | $0.7141 | $521.29 | 16 vCPU | 32 GiB | 80 GiB |

- The price is the size: it is the same whether the machine is idle or fully busy, and it is charged per second while the machine runs.
- Disk is provisioned, not used: you pay for the disk the size allocates, whether or not it is filled, at the storage rates above.
- `small` is the default size.
- Every size is a full machine with the same features, templates, and API. Only the resources change.
- Computers (desktops) come in `xs` through `xl`. A computer costs exactly what a sandbox of the same size costs: `xs` $0.0446 an hour, `small` $0.0893, `medium` $0.1785, `large` $0.3570, `xl` $0.7141. Their disks are larger: `xs` 10 GiB, `small` 20 GiB, `medium` 40 GiB, `large` 80 GiB, `xl` 160 GiB.

Apps and databases are billed on what they use, and never more than the price of their size in this table.
A small app that sits idle most of the day pays far less than the table; a busy one pays the table price at most.
Egress is $0.054 per GB, with no free allowance.
Deployment builds are free.
Your plan sets the largest size you can pick, never the price.

## Apps, services and databases

Apps, services and databases are billed on what they use, per second, and never more than the price of their size.

| What | Price |
|---|---|
| CPU actually busy | $21.915 per vCPU-month |
| Memory actually used | $10.953 per GiB-month |
| Idle CPU, unused memory, a stopped service | not billed |
| Most you can pay for a size, per second while it runs | the size price in the table above (`xs` $0.0446 an hour, `small` $0.0893, `medium` $0.1785, `large` $0.3570, `xl` $0.7141) |

- A small app that sits idle most of the day pays far less than its size price; a busy one pays the size price at most. The size is the cap, not the bill.
- Managed databases follow the same rule on their own sizes (never more than the sandbox price of the same size), plus disk at $0.162 per GiB-month.
- Running disk is $0.09 per GiB-month and egress $0.054 per GB. Builds are free.
- `GET /api/v1/pricing` lists each size under `products.machines.app.sizes` and `products.machines.database.sizes` with `usd_per_hour` and `usd_per_month`, and `price_kind` `maximum`.

## Plans and concurrency

| Plan | Price | Included usage each month |
|---|---|---|
| Free | $0 | $1 |
| Developer | $20/mo | $20 |
| Business | $500/mo | $500 |
| Enterprise | $2,000/mo | $2,000 |

Limits per service and per account are on [Plans and limits](/docs/plans-and-limits).

- The plan price comes back as usage: Business is $500 for $500 of usage, and every credit spends at the prices above.
- Plans never change the price of a size, only the caps.
- Top up any time from $1 to $5,000. Purchased credits never expire; recurring plan credits expire a month after they are granted.
- Start free with $1 of usage every month. Paid plans pay as you go. Free stops at its $1 unless you buy credits. Add credits any time.
- When your balance reaches $0, running computers and sandboxes pause and resume on their own when you add credit; databases and deployments are stopped and may need a manual restart after you add credit. Nothing is deleted. New machines start only while you have credit.
- Starting a sandbox is an API call, so it is rate-limited per API key (60 to 30,000 requests per minute by tier) and per plan. See [Rate limits](/docs/rate-limits).

## Get started

- [Create a free account](https://miosa.ai/register): Free includes $1 of usage every month. No card needed.
- [Pick a plan](https://miosa.ai/billing): Free works for a first try. Choose or change your plan in Billing.

Already signed up? [Sign in](https://miosa.ai/login).

## Limits

| Limit | Value |
|---|---|
| Concurrent sandboxes | See [Plans and limits](/docs/plans-and-limits) |
| Sandbox lifetime (`timeout_sec`) | 1 to 86,400 seconds, default 3,600 |
| Idle auto-pause | 30 minutes for accounts created on or after October 11, 2026; wakes on the next request |
| Exec | 30 seconds per call by default; detached jobs have no limit |
| Run code (`/code/run`) | 300 seconds maximum |
| File upload | 500 MB per write |
| Browser token | 1 hour by default, 60 seconds to 7 days |
| Named snapshots | 10 free per organization, then $1.70 a month each |

Cap one of your own end users, below the organization limit, with the [Quotas API](/docs/api-reference/quotas) (`max_sandboxes`, `max_concurrent`, `max_storage_gb`, `max_credit_cents`).

## Compared to others

The same machine, **4 vCPU and 8 GiB of RAM, running**, at each provider's public list price, checked 2026-10-20.
Each price is followed by its multiplier against MIOSA.

| Provider | 4 vCPU + 8 GiB / hour | vs MIOSA |
|---|---|---|
| **MIOSA** | **$0.1785** | **1x** |
| Railway | $0.2223 | 1.2x |
| Freestyle | $0.2645 | 1.5x |
| E2B | $0.3312 | 1.9x |
| Vercel † | $0.6816 | 3.8x |

† Bills active CPU, not allocated CPU.

MIOSA's figure is compute only; disk is metered separately at the rate above.
Several providers meter storage and egress on top of compute, so a real monthly bill can differ from the compute line.

MIOSA is below every provider listed here for this machine.
For capabilities and measured performance, see [MIOSA vs other platforms](/docs/compare) and the [compute provider comparison](/docs/integrations/comparison).

Sources, each checked 2026-10-20: [Freestyle](https://www.freestyle.sh/pricing), [E2B](https://e2b.dev/pricing), [Railway](https://railway.com/pricing), [Vercel Sandbox](https://vercel.com/docs/vercel-sandbox/pricing).

### Why some providers list a lower hourly price

Most sandbox providers sell the machine, and that is where their product ends.
You then stitch together other services for hosting, databases, agents, tenants and billing.

MIOSA prices the machine as part of a whole platform, so one account covers what you need to build and run a full-stack, multi-tenant application:

- [Deployments and Docker Host](/docs/docker-host/overview) to publish what you build to a stable URL with your own domain, versions and rollback.
- [Agents](/docs/guides/agent-operating-guide) with harnesses and approvals, running inside your machines.
- [Managed data](/docs/data/overview): Postgres, Redis, MySQL and Qdrant.
- [Organizations, workspaces and the white-label kit](/docs/platform/overview) to run it for your own team or sell it as your product.

If you only need a raw sandbox and nothing around it, a sandbox-only provider may fit.
If you want to build, deploy and run a product on one platform, that is where MIOSA's value is.

## Performance

MIOSA in the independent ComputeSDK benchmark suite. Every run is a committed JSON file, so each number can be checked against the repository.

| Benchmark | MIOSA | Field | Date |
|---|---|---|---|
| Burst startup: 100 concurrent creates, timed to the first command | 57.09 ms median | #1 of 32 | 2026-10-23 |
| DAX build: clone, install, typecheck in a fresh sandbox | 38.4 s | #1 of 16 completed | 2026-08-21 |
| DAX build, latest completed run | 39.2 s | #2 of 20 | 2026-10-02 |

See [Benchmarks](/docs/benchmarks) for every run and the raw results.
A cost per run is not published, because the benchmark sandbox size is not part of the result; size a run from the rates in [Worked examples](/docs/pricing#worked-examples).

## Worked examples

| You run | Per month |
|---|---|
| 1 `small` sandbox, 2 hours a day (60 h) | 60 x $0.089262 = **$5.36** |
| 1 `small` sandbox, always on (730 h) | 730 x $0.089262 = **$65.16** |
| 1 `medium` sandbox, 8 hours a day (176 h) | 176 x $0.178524 = **$31.42** |
| 1 `large` sandbox, always on (730 h) | 730 x $0.357048 = **$260.65** |
| 10,000 one-minute jobs on `small` | 166.7 h x $0.089262 = **$14.88** |
| 10 GiB kept paused for a month | 10 x $0.018 = **$0.18** |
| 10 GiB of disk on a running `small` sandbox, always on (730 h) | 10 x $0.09 = **$0.90**, on top of the $65.16 |

Compute for these examples only; running disk is metered on top at $0.09 per GiB-month.

## For the API

- `GET /api/v1/pricing` returns the price book this page is built from: for each product a list of sizes with `usd_per_hour` and `usd_per_month` (730 hours), the per-resource rates, and the plans. Sandboxes and computers have `price_kind: "flat"`; apps and databases have `price_kind: "maximum"`, the most they can cost for that size. `GET /api/v1/compute/pricing` serves the same body.
- `GET /api/v1/credits/balance` reports the balance in credits, with lifetime earned, lifetime spent, and the nearest expiry. One credit is $0.01, so 100 credits is $1.00.
- A sandbox or computer is billed the price of its size per second while it runs, with a 1-second minimum, and settled against the balance every 5 minutes. Apps and databases are billed on measured CPU and memory use, capped per second at the price of their size.
- The [Usage Rollup API](/docs/api-reference/usage-rollup) aggregates by `external_user_id`, `external_project_id`, or `workspace_id`, so a rollup maps onto your own billing.

> **Tip: One price per size for every plan**
>
> Every plan pays the same price for a size. Plans only change the caps and the concurrency, never the price.

## MIOSA AI pricing

MIOSA AI bills in wallet credits where 1 credit is 1 cent.
Each model has its own input and output price per 1M tokens, listed with real model names on the [pricing page](https://miosa.ai/pricing#ai-models) and under `ai.models` in `GET /api/v1/pricing`.
The default model, `kimi-k2.6`, is also reachable as `optimal`.
`deepseek-v4.1-flash` is the low-cost model and is also reachable as `optimal-fast`.
Every chat has the full context of its model, with automatic compaction when it fills.
See [Optimal](/docs/optimal) for how chats work.
