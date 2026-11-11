---
slug: "known-limitations"
title: "Known Limitations"
description: "An honest list of what doesn't work yet, what's in progress, and what's intentionally out of scope. Updated continuously."
section: "Optional"
order: 17032
---

We document what doesn't work, on purpose. If you hit something here, it isn't a bug in your code. It is a known gap. For release stages and the roadmap, see [Availability & Roadmap](/docs/availability); this page lists the concrete gaps and gotchas you can hit today.

> **Note**
>
> Last updated: 2026-10-11. We update this page on every release.

## Sandboxes `Public V1`

- **Availability is catalog-driven**. A named size, including `xs`, can be selected only when the template catalog and tenant policy admit it. A published contract is not a guarantee of immediate capacity in every environment.
- **The default is `small`**. It resolves to 2 vCPU, 4096 MiB RAM, and 10240 MiB disk. Raw CPU, memory, and disk fields must form a complete exact match for a published contract.
- **Timeouts are bounded**. `timeout_sec` defaults to 3600 and has a maximum of 86400. The extend endpoint replaces the timeout instead of adding duration. `always_on` is policy-dependent.
- **Usage can be incomplete**. Active CPU and network values can be `null` with `measurement_status` set to `unavailable`, or retained with status `stale`. Clients must not treat unavailable values as zero or stale values as current.
- **Snapshots are supported**. Public V1 includes customer-managed snapshot create, list, show, delete, and restore (`/sandboxes/{id}/snapshots` and `/sandboxes/{id}/restore/{snap_id}`), plus pause, resume, and fork.
- **AWS BYOC sandbox capacity is a gated private preview**. Managed and enabled BYOC environments share the sandbox contract, but BYOC placement is unavailable until the customer environment is accepted.
- **No GPU sandboxes**. The public sandbox create contract does not accept, persist, or schedule GPU requests; GPU runtime infrastructure exists behind a feature flag but is not productized, and no GPU rate is billable.
- **No nested virtualization**. KVM and QEMU cannot run inside a sandbox.

## Deployments `Phase 3/4 complete; dynamic runtime in hardening`

- **Custom domain apex records require ALIAS / ANAME**. DNS providers that do not support flattening, such as GoDaddy, require a workaround.
- **No edge functions yet**. Only origin compute is available.
- **Rollback is per-deployment, not per-route**. You cannot pin one URL path to version 1 and another to version 2 within the same deployment.
- **Containerized deployments publish through Docker Host**. Every sandbox ships Docker with the `docker compose` and `docker buildx` plugins, so you can build and test a container image there. A durable deployment of a containerized app is published through [Docker Host](/docs/docker-host/docker-deploy); the standard runtime builder builds from source and does not build OCI images into a deployment.
- **Railpack / Nixpacks framework detection** exists but dynamic generation from buildpacks is not complete.
- **Docker Host is a separate runtime path**. It publishes into the workspace Docker Host appliance and proves readiness through the `docker_deploy_app` row, container route, host health, and public URL. Standard MIOSA deployments and Docker Host deployments share the deployment API surface, but they do not use the same runtime substrate.

## Data Services

- **Managed Postgres** `Beta`. Provisioning lifecycle and credential injection exist. PgBouncer connection pooling and off-box backup drills are in progress.
- **Redis** `Beta`. Single-node only. Multi-zone replication is planned.
- **Object Storage** `Beta`. S3-compatible API is live; pre-signed browser uploads work. CDN distribution and geo-replication rollout are operational work in progress.
- **Auth-as-a-Service** `Beta`. Per-project email/password signup, login, JWT verify, password reset, and email confirmation endpoints are live. Refresh tokens, magic links, OAuth providers, MFA, and passkeys are still planned.
- **Volumes** `Beta`. Persistent local-backend volumes can be created, attached to computers, detached, deleted, and metered. Local volumes live on one host, so cross-host attach returns an error rather than moving the volume automatically.
- **Usage / audit query views grouped by `external_workspace_id`** have no API view. Raw audit events exist; aggregate them per workspace in your own code.

## Computers (Desktop) `GA`

- **Headed-only**: no headless mode for Computers. You boot a real Xfce desktop. For headless workloads, use Sandboxes.
- **Browser cookies do not persist** across sessions by default. Use a volume mount or browser profile snapshot to carry cookies across reboots.
- **No multi-monitor virtualization**. Single 1920 × 1080 frame buffer.
- **Audio not exposed**. The stream is video-only; system audio is dropped at the host.

## Computer-Use API

- **No managed agent loop**. You bring your own agent (Anthropic SDK, OpenAI computer-use API, or custom). A built-in `computer.prompt()` convenience will ship when the hosted Optimal session endpoint is publicly available.
- **Approval gating is not enforced server-side**. The approve / reject action protocol is a client-side convention only. Gate sensitive operations in your own agent code.
- **Computer sub-resources** (auto_stop, vnc_credentials, apps, stream_token, metrics, ports, volume attach/detach, clone) have full coverage in the Python SDK. TypeScript, Go, and Java parity is targeted Q3 2026.

## SDKs

| SDK | P1 + P2 resources | P3 / P4 resources | Computer depth |
|---|---|---|---|
| Python | Full | Full | Full |
| TypeScript | Full | Full | Partial |
| Go | Full | Partial | Partial |
| Java | Full | Partial | Partial |
| Elixir | Full | Partial | Partial |

- **Deployment proof parity is still uneven**. CLI, TypeScript SDK, Python SDK, and Go SDK expose proof checks. Java and Elixir proof helpers are still pending.
- **`embeddings`, `provider_defaults`, `benchmarks`, `command_center`, `community`, `email`, `builder_sessions`, `snapshots_standalone`** are Python-only today. Other SDK languages are pending.
- **Python sync client uses long-poll fallback** for streaming events. Use the async client or the TypeScript SDK for proper SSE.
- **Tab-synced language preference** persists in `localStorage`, not your user account. Switching language on one device does not propagate to another.

## API

- **No GraphQL endpoint**. REST + SSE by design.
- **Webhook delivery retries run for roughly 15.5 hours (within a 24-hour window)**. After eight attempts the event is dropped. Use the audit log to backfill missed events.
- **Rate limits**: 60 to 30,000 requests per minute per API key by tier, and 2,500/25,000/100,000 per minute per workspace on Developer/Business/Enterprise. Higher limits are available on request.
- **Browser tokens have a 1-hour default TTL** (60 seconds to 7 days). Refresh server-side; do not attempt client-side extension.
- **One sandbox per browser token**. A single token cannot scope across multiple sandboxes.

## Pricing and Billing

- **No per-tenant usage budgets yet**. You set a workspace cap today; tenant-level caps tied to `external_workspace_id` roll out with the billing attribution extension.
- **All traffic is billable** against the account’s spendable credit balance today. See [Availability & Roadmap](/docs/availability) for test-mode keys.

## Observability

- **No native log aggregation across workspaces**. Query per-resource today.
- **No OTLP trace export yet**. The `/events` SSE streams provide raw events; bring your own collector.

## See Also

- [Changelog](/docs/changelog) - what we just shipped
- [Concepts](/docs/concepts) - the resource model
- Contact platform@miosa.ai if a limitation here is blocking you. We prioritize the roadmap by demand.
