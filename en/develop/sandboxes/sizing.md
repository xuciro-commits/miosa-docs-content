---
slug: "develop/sandboxes/sizing"
title: "Sizing and limits"
description: "Named sandbox resource contracts, raw resource overrides, how sandbox usage is billed, and the opt-in runner fast path."
section: "Sandboxes"
order: 2006
---

Every sandbox runs on a named resource contract.
Pick a `size` by name; the raw CPU, memory, and disk fields are compatibility inputs that must exactly match a published contract.

## Sizing reference

| Size name | vCPU | RAM | Disk | Notes |
|---|---|---|---|---|
| `xs` | 1 | 2 GB | 10 GB | Lightweight scripts |
| `small` | 2 | 4 GB | 10 GB | **Default for agent and build workloads** |
| `medium` | 4 | 8 GB | 20 GB | Larger builds and parallel tests |
| `large` | 8 | 16 GB | 40 GB | Heavy test suites |
| `xl` | 16 | 32 GB | 80 GB | Intensive parallelism |

Use the default `small` contract for Claude Code, Codex, package installation, ordinary application builds, and agent execution.
Choose `medium` only when measured CPU, memory, or disk requirements exceed `small`.
Raw resource fields are compatibility inputs and must exactly identify a published named contract.

**Raw resource overrides** (still accepted):

| Field | Default | Validation range |
|---|---|---|
| `cpu_count` | 2 | 1-16 |
| `memory_mb` | 4096 | 2048-32768 |
| `disk_size_mb` | 10240 | Must match the selected named contract |
| `timeout_sec` | 3600 | 1-86400 |
| `idle_timeout_sec` | 0 (1800 on accounts created on or after October 11, 2026) | 0-86400 |
| `always_on` | `false` | Not required for most agent work |
| `persistent` | `true` | Preserve the disk across stops and resumes (survives stop, not host loss) |

Set `always_on=True` to disable `timeout_sec` enforcement entirely. Useful for long-lived development environments or hosted IDEs. Set `idle_timeout_sec` to stop an abandoned persistent session after a period of inactivity. Set `persistent=False` only for one-off jobs where the filesystem should be discarded on stop or timeout.

**Idle auto-stop default.** New accounts (created on or after October 11, 2026): sandboxes without `idle_timeout_sec` stop after 30 minutes idle and wake on the next request. Set `idle_timeout_sec` to `0` to disable. Existing accounts keep their current behavior (idle timeout off unless set).

## Configuration vs per-sandbox parameters

Platform configuration defines what can be selected: template aliases, immutable image generations, named resource contracts, readiness, placement policy, and tenant limits.
Customers inspect that configuration with `GET /api/v1/templates`; applications do not send internal placement, host, or provider fields.

Per-sandbox parameters select behavior for one sandbox.
They include `template_id`, `size`, `timeout_sec`, `always_on`, `persistent`, and `idle_timeout_sec`.
Prefer a named `size` over raw CPU, memory, and disk fields.
If compatibility fields are used, `cpu_count`, `memory_mb`, and `disk_size_mb` must be supplied together and exactly match one published contract.

The default is `small`: 2 vCPU, 4096 MiB RAM, and 10240 MiB disk.
`xs` is available for lighter work when the template catalog reports it for the selected template and current readiness permits admission.
Every sandbox response includes a versioned `resource_contract`, such as `sandbox/small@v1`, so clients can record the resolved resources instead of assuming a size name will never evolve.

```json
{
  "template_id": "miosa-sandbox",
  "size": "small",
  "timeout_sec": 3600,
  "persistent": true,
  "idle_timeout_sec": 0
}
```

## Resizing a running sandbox

A running Firecracker (`standard`) sandbox can be resized in place: vCPU, memory, or both, without destroying or recreating it.
The filesystem under `/workspace` is preserved.
Sizes are named contracts, so a resize moves a sandbox between published contracts rather than to an arbitrary value.

Read what the sandbox has and what it can change live, without changing anything.
Every sandbox reports its own capabilities, so a client acts on the answer instead of assuming:

**REST**

```http
GET /api/v1/sandboxes/:id/resources

{ "id": "...",
  "resources": { "cpu": 2, "memory_mb": 4096 },
  "capabilities": { "live_cpu": true, "live_cpu_max": 2,
                    "live_memory": false, "live_memory_max_mb": null,
                    "cpu_mechanism": "cgroup_quota", "runtime_engine": "firecracker" } }
```

**TypeScript**

```typescript
const r = await sandbox.resources();
```

**Python**

```python
r = sandbox.resources()
```

Change it, sending only the fields you want to change:

**REST**

```http
PATCH /api/v1/sandboxes/:id/resources
{ "cpu_count": 4, "memory_mb": 8192 }
```

**TypeScript**

```typescript
await sandbox.resize({ cpuCount: 4, memoryMb: 8192 });
```

**Python**

```python
sandbox.resize(cpu_count=4, memory_mb=8192)
```

**CLI**

```bash
miosa sandbox resources my-box
miosa sandbox resize my-box --cpu 4 --memory 8192
```

What is live today:

- **vCPU** is live on a running Firecracker sandbox, up to the count the sandbox booted with, and only on a host with the vCPU controller enabled.
  The guest keeps the cores it booted with; the host caps the time they may use with a cgroup quota, so a lower vCPU count throttles rather than removes cores.
- **Memory** is live on a running Firecracker sandbox booted with a hotpluggable region (virtio-mem): growing plugs memory into the guest, shrinking unplugs it, and memory never goes below the booted size.
  `live_memory_max_mb` is the ceiling for that sandbox, 16384 MiB by default.
  Live memory is rolling out across the warm pool: a sandbox from an older warm slot reports `live_memory: false` until that slot is rebuilt, and so does a sandbox restored from a snapshot.
- **SOMA** sandboxes apply vCPU and memory live on their own host, up and down (vCPU by cgroup quota, memory by virtio-mem), where the platform has enabled it for that host.

Read `capabilities` before a resize, and again from the resize response.
`live_cpu`, `live_memory`, and their maxima are per sandbox, per host, and per image generation; the `*_reason` field says why a change is not live when it is not.

A change that cannot be applied live is reported `requires_restart`: the request answers `409 RESIZE_REQUIRES_RESTART`, nothing is applied, and nothing is stored as pending.
A request above your plan cap answers `422 PLAN_CAP_EXCEEDED`; a SOMA sandbox or a host without the controller answers `409 RESIZE_UNSUPPORTED`.

Send `"restart": true` to boot the same disk at the new size instead.
This is a recreate, not an in-place change: the filesystem survives, the new sandbox has a new id, the processes of the old one are gone, and the source is left paused and billed as a paused sandbox until you destroy it.
The response is `202` with `restarted: true`, `new_sandbox`, and `source_sandbox`.
SOMA sandboxes have no host disk to boot from, so they are not recreated this way.

## Billing notes

Billing accrues from `state = "running"` until `state = "stopped"`, `state = "paused"`, or `state = "destroyed"`.

- **Running**: billed per second at the price of its size, the same on every plan and whether the machine is idle or busy (for example `small` is $0.0893 an hour)
- **Stopped**: billed at the cold storage rate (disk GiB-second)
- **Paused**: billed at storage rate only (disk GiB-second)
- **Destroyed**: billing stops immediately

Template readiness and available sizes are reported by the catalog and can vary by environment.

## Runner endpoints (opt-in preview)

The SOMA one-hop runner is an opt-in fast path for `create`, `exec`, and `destroy`: your SDK talks straight to the runner instead of going through the control plane first. Everything documented here keeps working unchanged whether or not you opt in.

- `https://run-us.miosa.ai` works for every call. Opting in sends requests there instead of `https://api.miosa.ai`; EU is on the roadmap, but keys do not carry a region yet. This endpoint is unrelated to a sandbox's placement [region](/docs/api-reference/regions) (`us-west`, `us-east`, `us-mia`).
- A request for a sandbox that lives on a different host inside the runner fleet is forwarded internally; you almost always get an answer from `run-us.miosa.ai` directly (`421` is a rare fallback if that internal forward fails, naming the correct URL as `runner_url`). For lower latency, every `create` response also names that sandbox's own runner URL as a `Soma-Runner-Url` response header (`https://<tag>.run-us.miosa.ai`); official SDKs read it and call that URL directly from then on. This is an automatic optimization, not something you need to do yourself.
- `create` on the runner only serves the default template, `xs` size, non-persistent sandboxes. A request outside that (a custom template, a snapshot, a custom `cwd`, `persistent: true`, or a larger size) answers `400 RUNNER_UNSUPPORTED_REQUEST`; official SDKs send that create to `https://api.miosa.ai` automatically instead.
- The runner only creates non-persistent sandboxes, so the idle auto-stop default does not apply here: idle timeout stays off unless you set `idle_timeout_sec` (0-86400 s). `timeout_sec` (default 3600, max 86400) caps lifetime, and `always_on=True` disables the timeout entirely for long-lived sandboxes and hosted IDEs.
- `exec` defaults to a 30 second timeout, same as the control plane. `POST /api/v1/sandboxes/{id}/exec/stream` streams stdout/stderr as server-sent events, ending with an exit event.
- `destroy` responses include `cpu_ms` and `lifetime_ms` for this sandbox's usage.
- `429 runtime_busy` (with `Retry-After: 0`) means this runner is at its admission limit; retry the request. `429 rate_limited` is a per-key or per-tenant rate limit, same semantics as the control plane. `503 feed_stale` means the runner's key and policy data is too old to accept new creates; its existing sandboxes keep working.
- Runners serve HTTP/3 (QUIC) first and fall back to HTTP/2 automatically if QUIC is blocked on your network. Official SDKs handle the fallback for you.

A raw HTTP client can use either the regional URL or the per-sandbox URL named in the `Soma-Runner-Url` header:

```bash
curl -i -X POST https://run-us.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"template_id": "miosa-sandbox"}'
# Soma-Runner-Url: https://3.run-us.miosa.ai

curl https://3.run-us.miosa.ai/api/v1/sandboxes/3a1b2c3d-1111-2222-3333-444444444444 \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Every runner response adds a `Server-Timing` header breaking `auth`, `pool`, and `exec` time down in milliseconds.

### Errors specific to the runner path

| Status | Code | Cause |
|--------|------|-------|
| 401 | `unauthorized` | The key is unknown or revoked in this runner's in-memory table. |
| 403 | `forbidden` | The tenant is not enabled for this path, is suspended, or the project is not allowed. |
| 400 | `RUNNER_UNSUPPORTED_REQUEST` | This create shape is not served by the runner (see above). Official SDKs retry it against `https://api.miosa.ai`. |
| 429 | `rate_limited` | Per-key or per-tenant rate limit, same semantics as the control plane. |
| 429 | `runtime_busy` | This runner is at its admission limit (`Retry-After: 0`). Retry the request; `run-us.miosa.ai` resolves to every runner in the region, so a retry can land on a different one. |
| 503 | `feed_stale` | This runner's key and policy data is older than its freshness limit (`Retry-After: 0`). Creates are refused; its existing sandboxes keep working. |
| 421 | `misdirected` | Rare: the internal forward to the sandbox's own host failed. The body names the correct URL again as `runner_url`. Follow it once; do not retry the original URL. |

The full endpoint contract (request and response bodies) is unchanged by runner mode; only the host you send them to changes. See the [Sandboxes API](/docs/api-reference/sandboxes) for the control-plane reference.

> **Note**
>
> Runner mode is an opt-in preview rolling out gradually by tenant and host. Until you opt in, every request keeps going through the control plane exactly as documented above.
