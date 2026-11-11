---
slug: "docker-host/runtime-instances"
title: "Dynamic Runtimes"
description: "Dynamic deployments: production runtimes started from a release, health-checked, and reconciled by MIOSA."
section: "Services & projects"
order: 1031
---

A **Runtime Instance** is a running production runtime that serves a dynamic deployment. Static deployments do not have runtime instances; they are served from immutable static artifacts.

> **Note: Phase status**
>
> The caller-facing contract is stable: dynamic deployments start production runtimes, wait for health checks, and only promote when the new version is ready. You do not need to know how MIOSA places or reconciles those runtimes.

## What a runtime instance is

A runtime started from the customer's release. The release is the customer's app, fully packaged. The runtime:

1. Boots in under a few seconds.
2. Receives `PORT` (the release port, unless you set your own), `DATABASE_URL`, `AUTH_URL`, and other env vars injected by MIOSA at boot.
3. Starts the configured `run_command`.
4. Listens on `port`.
5. Responds to `GET health_check_path` with 2xx.
6. Joins the deployment's backend pool; the router starts sending it traffic.

When a runtime instance dies (crash, host failure, configured restart), MIOSA's reconciler notices and schedules a replacement on another healthy host.

## Where it runs

Each runtime instance runs in its own microVM on a MIOSA-managed host.
Runtimes of different customers are segmented from each other on the network, and a runtime can reach only the services you have linked to it plus the public internet.
A restart of the host's agent does not stop a running instance.
The agent reattaches to it when it comes back.

## The port your app must use

MIOSA sets `PORT` to the release's port (the service `port`, `8080` by default) unless you set `PORT` yourself in the deployment's env vars.
Your app must listen on that port on all interfaces (`0.0.0.0`), not only on `127.0.0.1`.
MIOSA reaches it from inside the microVM and publishes it at the instance's address.
The address and port the API reports for an instance are where MIOSA's router reaches it, and are not always the runtime's private address.

## Next.js apps

Set `output: "standalone"` in `next.config`.
When the build produces `.next/standalone/server.js`, MIOSA copies `public` and `.next/static` beside it and runs:

```bash
HOSTNAME=0.0.0.0 node .next/standalone/server.js
```

A standalone release carries its own traced `node_modules`, so MIOSA does not reinstall dependencies in the runtime.
Without standalone output, a Next.js app starts with `next start` and needs the full build.

## Build output is required

A Next.js release must contain its production build: `.next/BUILD_ID` has to exist in the release.
A release that is only the source tree cannot run, because `next start` stops at once with `Could not find a production build`.
MIOSA refuses such a release at publish time and again at start time, with the error `build_output_missing`.
Fix it by running the build before the release is created, or by letting MIOSA build the app, and publish again.

## When a start fails

A runtime instance start fails, and the instance is removed, when any of these happens:

| Error | Meaning |
|---|---|
| `release_install_failed` | Downloading, verifying or unpacking the release failed. The error carries the exit code and the last output of the step. |
| `build_output_missing` | A Next.js release has no production build (see above). |
| `app_launch_failed` | The start command could not be launched. |
| `app_not_listening` | The app started but did not answer its health check on its port. |

For `release_install_failed`, `app_launch_failed` and `app_not_listening`, the deployment's error message includes the tail of the app's log, the processes running in the runtime, and the ports it is listening on, so you can see why it did not come up.
Read it first: the usual causes are an app that binds `127.0.0.1` instead of `0.0.0.0`, a wrong `port`, a missing dependency, or a missing build.
Because the failed instance is removed, a start that fails never serves traffic and the previous version stays live.

## What you control vs what MIOSA controls

| You | MIOSA |
|---|---|
| Release contents (your code + dependencies) | Where to schedule instances |
| `run_command` and `port` | When to start / stop / replace instances |
| `health_check_path` | Routing healthy instances to traffic |
| Desired replica count (eventually; defaults to 1) | Boot-time env var injection |
| Env vars set on the deployment | Telemetry, logs streaming |

You don't pick a host directly. MIOSA's scheduler matches your workload's requirements (CPU, RAM, region) against the fleet's free capacity.

## Lifecycle

pending goes to starting on scheduled; starting goes to healthy on checks pass; healthy goes to stopping on tear down; stopping goes to stopped.

starting goes to unhealthy on checks fail; unhealthy goes to stopping on replaced.

starting goes to failed on crashloop budget spent.

- `pending`: scheduled, not yet booting.
- `starting`: VM is booting; health check not yet passing.
- `healthy`: serving traffic.
- `unhealthy`: failing health checks; router has removed it from the pool. Reconciler decides whether to restart or replace.
- `stopping` / `stopped`: deliberately torn down.
- `failed`: terminal, having exceeded the crashloop budget. Reconciler stops trying.

## Health checks and the grace period

When a runtime instance starts, it has a configurable grace period (default 60s) to become healthy. During grace, repeated 5xx / connection-refused are tolerated.

After grace, the health check is enforced strictly:

- 3 consecutive failures → instance marked `unhealthy`, removed from routing.
- If a replacement is needed and starts healthily, the old instance is torn down.
- Repeated rapid failures invoke crashloop backoff: 1s → 2s → 5s → 30s → 5min between restart attempts.

## Blue/green during publish

When you publish a new dynamic version, MIOSA does NOT terminate the existing healthy instances first. The sequence is:

1. Start new runtime instances from the new release.
2. Wait until enough new instances pass health checks.
3. Update the router so the deployment's hostname points at the new instances.
4. Drain (stop accepting new connections) and tear down old instances after old in-flight requests complete.

If step 2 never succeeds, step 3 never runs. The old version stays live, the new version is marked `failed`.

## Replicas

By default, a dynamic service has 1 replica. Increase by setting `desired_replicas` on the service. The reconciler spreads replicas across hosts and across availability zones where possible.
```typescript
await miosa.services.update(serviceId, { desiredReplicas: 3 })
```

Traffic load-balances across all healthy replicas in round-robin (or hash-based if sticky sessions are configured).

## Resource sizing

| Spec | Default | Tunable per service |
|---|---|---|
| CPU | 1 vCPU | yes |
| Memory | 512 MB | yes |
| Disk | 1 GB ephemeral | yes |
| GPU | none | not available (GPU is not productized) |
| Outbound bandwidth | metered | yes |

Sizing is per-service, per-deployment. A worker service can be smaller than a web service.

## Logs

Runtime instances stream stdout/stderr to MIOSA's log plane. Logs are retained per the tenant's plan (default 7 days). Access them through:

- `GET /api/v1/deployments/:id/logs`: deployment-scoped, all services
- `GET /api/v1/services/:id/logs`: per-service
- `GET /api/v1/runtime-instances/:id/logs`: per-instance

Logs include structured metadata: `service_id`, `runtime_instance_id`, `version_id`, `host_id`. Filter by `external_workspace_id` to scope to a white-label customer.

## See also

- [Releases](/docs/docker-host/releases): what runtime instances boot from
- [Environments](/docs/docker-host/environments): variable / secret scope
- [Domains](/docs/docker-host/domains): how traffic finds your instances
- [Rollback](/docs/docker-host/rollback): what happens to old instances on rollback
