---
slug: "compare/cloudflare"
title: "MIOSA vs Cloudflare Sandboxes"
description: "A factual comparison of MIOSA and Cloudflare Sandboxes, using the independent ComputeSDK benchmark and Cloudflare Sandboxes's public documentation as of October 2026."
section: "Optional"
order: 17021
---

Compare

Cloudflare's Sandbox SDK and MIOSA both run untrusted code for AI agents. This page compares them on the independent ComputeSDK benchmark and on the capabilities MIOSA documents.

## What Cloudflare Sandboxes is

Cloudflare's Sandbox SDK runs sandboxed code execution on Cloudflare's edge, described in its public documentation at ([developers.cloudflare.com/sandbox](https://developers.cloudflare.com/sandbox/), as of October 2026).

## Measured performance

The independent ComputeSDK benchmark runs the same workload against every provider. That is the only apples-to-apples number here.

| Metric | MIOSA | Cloudflare Sandboxes | As of |
|---|---|---|---|
| Burst startup, median time-to-interactive | **57.09 ms (#1 of 32)** | 682.13 ms (#12) | 2026-10-23 |
| DAX full build, 2026-08-28 | **39.3 s (#1 of 19)** | no result | 2026-08-28 |
| DAX full build, 2026-10-23 | **38.8 s (#3 of 22)** | 155.9 s (#21) | 2026-10-23 |

Source: ComputeSDK, via [our benchmark page](/docs/benchmarks). Burst startup is the time from a create request to the first successful command inside a fresh sandbox, under 100 concurrent creates. Where a run is not listed, the provider did not appear in that run.

> **Note: Fair reading**
>
> Cloudflare's strength is the edge network and its integration with Workers. Where your architecture is already edge-first, that integration can outweigh a startup-time gap.

## Capabilities side by side

Each Cloudflare Sandboxes answer comes from Cloudflare Sandboxes's own documentation or pricing page, linked in the last column, fetched on 2026-10-23.
Each MIOSA answer links the page that documents it.
"Not documented" means the vendor's docs do not say, not that the feature is missing.

| Capability | MIOSA | Cloudflare Sandboxes | Source |
|---|---|---|---|
| Isolation | [Firecracker microVM by default](/docs/develop/sandboxes) | Containers in a Firecracker microVM | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/concepts/) |
| Sandbox lifetime | [Idle timeout, 1 hour by default and up to 24 hours, extendable; computers persist](/docs/develop/sandboxes/timeouts) | Stops after inactivity; inactivity timeout up to 6 hours | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/concepts/lifetime/) |
| Pause, snapshots, forks | [Yes: pause and resume, snapshots, restore and fork](/docs/develop/snapshots) | Filesystem snapshots (beta); no pause and resume | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/concepts/lifetime/) |
| Full Linux desktop and computer use | [Yes, Computers](/docs/computers/overview) | Not documented | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/llms.txt) |
| Public preview URL for a port | [Yes](/docs/develop/previews) | Yes, through your own Worker | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/previews/) |
| Deploy to a production URL | [Yes, Services and Docker Host, with releases and rollback](/docs/services/overview) | Yes, deployed as a Worker with wrangler | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/sdk/guides/deploy/) |
| Managed databases | [Postgres, MySQL, Redis, MongoDB, Qdrant](/docs/data/overview) | Not documented for Sandboxes | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/llms.txt) |
| SDKs | [TypeScript, Python, Go, Elixir, Java, Rust](/docs/sdks) | TypeScript | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/) |
| CLI | [Yes](/docs/cli) | wrangler; no sandbox-specific CLI | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/sdk/guides/deploy/) |
| MCP server | [Yes, hosted](/docs/mcp) | Not documented | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/llms.txt) |
| Bring your own cloud | [Yes, AWS and Google Cloud](/docs/infrastructure/byoc/overview) | Not documented; runs in your Cloudflare account | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/sandbox/) |
| Billing | [Per second while running, one price per size](/docs/pricing) | Containers pricing, in 10 ms steps | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/containers/platform/pricing/) |
| Regions | [us-east, us-west, us-mia](/docs/regions) | Not documented | [Cloudflare Sandboxes docs](https://developers.cloudflare.com/containers/platform/limits/) |

## Where MIOSA differs

These capabilities are documented for MIOSA on this site:

- **Persistent computers with a full Linux desktop**, not only headless sandboxes. See [Computers](/docs/computers/overview).
- **Deployments and Docker Host**, so a machine becomes a versioned URL or a containerized app. See [Deployments](/docs/docker-host/overview).
- **White-label platform**, so your customers see your brand and domains. See [White-label setup](/docs/platform/white-label).
- **One interface for everyone**: MCP, a CLI, and six SDKs. See [MCP](/docs/mcp), [CLI](/docs/cli), [SDKs](/docs/sdks).
- **Managed data** with connection injection: Postgres, MySQL, Redis, MongoDB, Qdrant. See [Data](/docs/data/overview).
- **Regions and OpenComputers** for machines you own. See [Regions](/docs/regions) and [OpenComputers](/docs/opencomputers).

## When to choose each

- **Choose Cloudflare Sandboxes** if you build on Cloudflare Workers and want code execution close to your edge compute and data.
- **Choose MIOSA** if you want a runtime that covers sandboxes, full computers, agent runs, deployments, and managed data in one place, with a branded tenant surface and the measured startup figures above.

## Sources

- Cloudflare Sandboxes public documentation: https://developers.cloudflare.com/sandbox/ (as of October 2026).
- ComputeSDK benchmark results, via [our Benchmark page](/docs/benchmarks) (2026-10-23).
- MIOSA capabilities, as documented on this site.

- [MIOSA vs Freestyle](/docs/compare/freestyle): The next comparison.
- [All comparisons](/docs/compare): Every provider MIOSA is measured against.
