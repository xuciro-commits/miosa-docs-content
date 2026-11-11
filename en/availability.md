---
slug: "availability"
title: "Availability & Roadmap"
description: "Current MIOSA product availability, what is generally available, what is in hardening, and what is planned next."
section: "Optional"
order: 17031
---

This page gives customers a clean read on what is ready for production today, what is available for selected partners, and what is still on the roadmap.

> **Note**
>
> Last updated: 2026-07-25.
> We update this page when a platform capability changes release stage.

## Release stages

| Stage | Meaning |
|---|---|
| **Generally available** | Supported for production use with documented APIs and normal support paths. |
| **Partner preview** | Available to approved customers while we finish polish, SDK parity, or operating runbooks. |
| **In hardening** | Built or partially available, but not ready to promise as a stable public surface. |
| **Planned** | On the roadmap, not available for production use yet. |

## Generally available

- **Sandboxes**: Create isolated Linux workspaces for code generation, build steps, test runs, long-running commands, file operations, and live previews.
- **Live previews**: Expose a running process on a public preview URL, with private preview tokens for embedded product experiences.
- **Publishing and rollback**: Turn a sandbox into a durable deployment version, promote it, list versions, and roll back when needed.
- **Managed data services**: Use Postgres, MySQL, Redis, Qdrant, object storage, per-project auth, and persistent volumes with credentials injected into runtime.
- **White-label platform**: Use external IDs, browser tokens, custom branding, preview domains, webhooks, and usage rollups to build MIOSA into your own product.

## Partner preview

These surfaces are being used with selected customers while we continue polishing public docs and SDK parity.

| Surface | Current scope |
|---|---|
| **Computers** | Full Linux desktops for browser automation, screenshot loops, and computer-use agents. |
| **Dynamic deployments** | Production server processes created from a sandbox publish. |
| **Docker Host** | Workspace-scoped container runtime for apps that need Docker, server processes, background workers, or database-backed production hosting. |
| **Tenant deployment domains** | Branded app domains for generated apps, separate from sandbox preview domains. Routing exists; tenant-level self-service remains operator-assisted while exact deployment domains are self-service. |
| **External computer providers** | Bring third-party computer providers into the same MIOSA computer abstraction. |
| **Evaluations** | Desktop and agent workflow benchmarks using reproducible tasks, traces, and scoring. |

## In hardening

| Area | Status |
|---|---|
| **Desktop computer provisioning** | Computers preserve disks and snapshots across stop, resume, and failed recovery. Start latency and stream readiness continue to be hardened. |
| **Dynamic runtime routing** | Dynamic runtimes keep the active healthy version serving while candidates build. Route evidence and automated recovery continue to be hardened. |
| **Docker Host fleet maintenance** | Host upgrades preserve deployment records and desired state, restore desired-running apps, and verify workloads before returning a host to active service. |
| **Customer-cloud regions** | AWS and Google Cloud foundations exist, but production placement remains gated on a real provider canary and per-host acceptance evidence. |
| **Managed data recovery** | Database IDs, credentials, volumes, bindings, and desired state survive host maintenance. Recovery visibility continues to be expanded across product surfaces. |
| **Deployment proof** | CLI, TypeScript SDK, Python SDK, and Go SDK expose proof checks. Java and Elixir parity is still in progress. |
| **SDK parity** | Python and TypeScript are deepest today. Go, Java, Elixir, and UI packages are being brought to the same coverage over time. |
| **Cross-workspace analytics** | Resource attribution is live. Higher-level dashboards and billing views are expanding. |

## What is live in Release 4

Recent platform changes customers can rely on now.
Each is a result, not a promise of future work.

| Change | Result |
|---|---|
| Disk corruption guard | Restoring a memory snapshot onto a disk that grew after the snapshot is refused instead of corrupting the filesystem. Live on the control plane and all hosts. |
| Guest isolation from the internal network | A sandbox cannot reach the platform's own network except the managed-database proxy range. The Firecracker rule ordering that stops an older allow rule from shadowing the deny is live fleet-wide, and SOMA already enforced it. |
| Preview fallback for white-label staff | Preview access for a white-label organization's own staff falls back to the authenticated console path instead of failing. |
| Disk larger than the pre-built slot | A request for a disk larger than the pre-built slot boots at the requested size instead of failing or returning the wrong size. |
| Learn credits cooldown | An expired Learn attempt is timed from its expiry, and an expired attempt with no answers no longer counts as a failure. |
| Function pool | The per-host function pool is deployed but dark: it is not a customer surface yet. |
| SOMA catalog accuracy | The runtime catalog no longer advertises pause or snapshots for SOMA, which has neither. |

## Planned

| Capability | Notes |
|---|---|
| Native GPU sandboxes | Targeted early 2027, for workloads that need accelerated inference or rendering. |
| Edge functions | Targeted Q1 2027, as a deployment target separate from origin runtimes. |
| Native nameservers | Targeted Q4 2026, for customers who want MIOSA to manage more of the domain lifecycle. |
| OTLP export | Planned for teams that want platform traces in their own observability stack. |
| Test-mode API keys | Planned to separate production billing from integration testing. |

## Deliberate product boundaries

MIOSA is focused on AI-generated software, isolated execution, live previews, desktop automation, white-label SaaS infrastructure, and durable deployment. It is deliberately not:

- **Managed Kubernetes**. Sandboxes and Deployments cover the workloads people would otherwise use K8s for.
- **Generic VPS hosting**. Compute is always tied to a tenant identity (workspace or `external_workspace_id`).
- **Cryptocurrency or blockchain primitives**.

## See also

- [Benchmarks](/docs/benchmarks) for verified performance results.
- [Evaluations](/docs/evaluations) for agent task scoring and desktop workflow benchmarks.
- [Provider comparison](/docs/integrations/comparison) for the compute-provider landscape.
- [Customer infrastructure](/docs/infrastructure/byoc/overview) for MIOSA-managed, connected-machine, AWS, Google Cloud, and hybrid placement.
- [Known limitations](/docs/known-limitations) for the concrete gaps and gotchas behind these stages.
- [Changelog](/docs/changelog) for shipped product changes.
