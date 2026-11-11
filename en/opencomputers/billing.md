---
slug: "opencomputers/billing"
title: "Billing and limits"
description: "What a plan allows on OpenComputers - hosts, containers, tunnels - and the operational caps on commands, transfers, backups, and tunnels."
section: "Computers"
order: 3023
---

Your machine is your own bill. You pay for the hardware, the power, the network, and the hosting, and MIOSA does not charge you for the machine.
MIOSA charges for platform usage at the rates on the [pricing page](https://miosa.ai/pricing); your plan sets limits, never the price.

This page covers the limits that a plan puts on OpenComputers, and the operational caps that apply to everyone.

## Plan limits

Plan limits are per organization.

| Plan | Hosts | Container and compose service slots | Tunnels per host |
|---|---|---|---|
| Developer | 3 | 2 | 0 |
| Business | 50 | 250 | 50 |
| Enterprise | 500 | 1000 | 500 |

Registering past a limit returns `402` with `error: "plan_limit_reached"`, plus the `limit` and `current` values.

A compose project counts its services against the same container budget, so a project with three services uses three slots.

> **Note: Read the limit from the error, not from this table**
>
> Plan limits are enforced by the platform and can change between releases.
> The authoritative value is the one in the `402` response, and the one the UI shows for your organization.

To raise a limit, contact support with your expected scale. See [Support](/docs/support).

## Operational limits

These apply regardless of plan.

| Limit | Value |
|---|---|
| Command length in a job | 1 to 8192 characters |
| Arguments per job | 256 |
| Working directory length | 4096 characters |
| Job timeout | 1 to 3600 seconds; default 30 |
| Exec stream timeout | 30000 ms |
| File transfer chunk | 4 MiB per read or write |
| Backup chunk | 8 MiB compressed |
| Backup compression level | 1 to 22; default 19 |
| Backup retention | 1 to 365 snapshots, or 1 to 3650 days |
| Tunnel frame | 4 MiB |
| Tunnel connect timeout | 10 s |
| Tunnel response timeout | 30 s |
| Forwarded request headers per tunnel request | 64 |
| Container stats tick | 10 s |
| Mesh size | 253 hosts |
| Tags per host | 20 |
| Device browser token TTL | 3600 s |

Job commands are also recorded in audit truncated to 512 characters, and environment variable names are recorded without their values.

## Counters you can read

The platform meters usage per resource, and the counters are on the records you already fetch.

| Counter | Where |
|---|---|
| Host egress and ingress bytes | `usage.lifetime_bytes_egress` and `usage.lifetime_bytes_ingress` on the host record |
| Tunnel requests and bytes | `request_count`, `lifetime_bytes_ingress`, and `lifetime_bytes_egress` on the tunnel |
| Snapshot size | `total_bytes` and `compressed_bytes` on the snapshot |
| Container resource use | `last_cpu_percent`, `last_mem_mb`, and `last_stats_at` on the container |

For the platform-wide picture, use the [Pricing and limits](/docs/pricing) page and the [Usage Rollup API](/docs/api-reference/usage-rollup).

- [Pricing and limits](/docs/pricing): Billable states, machine sizes, and the platform-wide caps.
- [Containers](/docs/opencomputers/containers): Run and watch containers against the container slot budget.
- [Backups and snapshots](/docs/opencomputers/backups): Retention, compression, and restore.
- [Support](/docs/support): Ask for a higher limit with your expected scale.
