---
slug: "develop/sandboxes/lifecycle"
title: "Sandbox lifecycle"
description: "The sandbox state machine from creating to destroyed, what each state means, what stop keeps, what resets the idle clock, and how to stop, pause, resume, and restart."
section: "Sandboxes"
order: 2004
---

A sandbox moves through a small set of states.
This page is the map: what each state means, what triggers a transition, and how to stop and restart compute without discarding the disk.

## Lifecycle

create() creates creating; creating goes to running on ready.

running goes to stopped on stop(), and back to running on resume().

running goes to paused on pause(), and back to running on resume().

running, stopped or paused goes to destroyed on destroy().

creating or running goes to error on failure; error goes to destroyed on cleanup.

| State | Meaning |
|---|---|
| `creating` | VM is being claimed, restored, and prepared to accept commands |
| `running` | Command-ready: exec, files, previews, terminal, and port exposure work |
| `stopped` | The machine is released and the disk is saved; files survive, processes do not |
| `paused` | The machine is suspended in place with memory and processes intact; only on a runtime that advertises pause |
| `destroyed` | Terminal: resources freed, ID unusable, saved state removed |
| `error` | Boot or runtime failure; check `sbx.data["metadata"]` for `last_error` |

`paused` exists only on a runtime that advertises pause.
Firecracker advertises it today.
SOMA does not: it has no pause, and it does not keep state across a stop yet, so a SOMA sandbox is never `paused`, and a disposable SOMA sandbox is deleted when its timeout expires.
Snapshots are a separate resource, not a sandbox state.

Activity that resets the idle clock: exec calls, file operations, preview HTTP traffic, terminal stdin.
The timeout is idle-anchored: it counts from the last activity.
Once `timeout_sec` elapses with no activity, a persistent sandbox is stopped.
A non-persistent sandbox (`persistent: false`) is destroyed, and its disk is discarded.

`idle_timeout_sec` stops a persistent sandbox after a period of inactivity.
New accounts (created on or after October 11, 2026): sandboxes without `idle_timeout_sec` stop after 30 minutes idle and wake on the next request.
Set `idle_timeout_sec` to `0` to disable.
Existing accounts keep their current behavior (idle timeout off unless set).

`resume()` is not the only way back out of `stopped` or `paused`: a persistent sandbox **wakes on request**.
A request that arrives while it is stopped or paused (for example, to its preview URL) resumes it and is held for up to 10 seconds, then answers `503` with `Retry-After` if it is not running yet.
Exec, file operations, and deploys on a stopped or paused persistent sandbox resume it first and then run.
A non-persistent sandbox is never auto-resumed: the call returns `409`.

## What stop keeps

Stop saves the disk and releases the machine, and moves the sandbox to `stopped`.
Files, installed packages, Docker data, and enabled services survive a stop.
Running processes never survive a stop, on any runtime.
Resume after a stop behaves like a server that restarts: start your processes again, or make them declared services.

Pause is the explicit verb for keeping processes.
It is available on Firecracker only.
A paused sandbox stays resident for at most 30 minutes, with memory and processes intact.
After 30 minutes it behaves like a stop: the disk is kept and processes are not.
A memory checkpoint is an accelerator, never the truth: if it cannot be restored, resume falls back to the disk.

Disk size is a fixed class (10, 20, or 40 GiB) chosen at create.
It can grow only as a parameter of `resume` or `fork`.
A request to shrink is refused with `409` and the sandbox is left untouched.
Disk growth is not available yet on every runtime.

Persistent means the sandbox survives a stop.
It does not mean the sandbox survives the loss of its host.
The `durability` field on the sandbox record states how many hosts hold the saved state: `local_only`, `replicating`, or `replicated`.
Every Firecracker sandbox exposes `durability`.
It is `local_only` until a stopped sandbox's copy has been verified off-host, and then `replicated`.
Off-host replication is rolling out on a QA canary first, so expect `local_only` for most sandboxes today.

See [Persistence, pause, and forks](/docs/develop/snapshots) for the full pause/resume/fork flow.

## Readiness and billing

A sandbox is command-ready only in the `running` state. Two signals tell you when it gets there:

- The `ready` boolean on the sandbox record. `true` means the VM session is prepared to accept exec, files, previews, and terminal calls.
- The `state` field. Poll `GET /api/v1/sandboxes/{id}` until `state` leaves `creating` and becomes `running` (or surfaces `error`).

The fused [run endpoint](/docs/develop/sandboxes/create) owns this wait on the server, so agents that create-and-exec do not need a client poll loop.

Which states cost what:

| State | Billing |
|---|---|
| `creating` | Preparing resources; not yet command-ready |
| `running` | Billed per second at the price of its size (the same on every plan) |
| `stopped` | No compute; cold storage rate only (disk GiB-second) |
| `paused` | No compute; storage rate only (disk GiB-second) |
| `destroyed` | Billing stops immediately |

See [Sizing and limits](/docs/develop/sandboxes/sizing) for the full billing notes and [Timeouts and auto-stop](/docs/develop/sandboxes/timeouts) for what happens at the deadline.

## Pause and resume

`POST /stop` and `resume()` release the machine and bring it back on the same disk.
`pause()` and `resume()` suspend the machine in place and keep its processes, on a runtime that advertises pause (Firecracker only, for at most 30 minutes before it behaves like a stop).

**Python**

```python
sbx.pause()
# Firecracker only: state is "paused" and processes are kept for up to 30 minutes

sbx.resume()
# state returns to "running"
```

**TypeScript**

```typescript
// Firecracker only: keeps processes for up to 30 minutes
await sbx.pause();
await sbx.resume();
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/stop \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'

curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/resume \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Restart

`POST /sandboxes/{id}/restart` (`miosa restart`) is a stop followed by a resume of the same disk, under the same id.
Files, packages, Docker data and enabled services survive; running processes do not, and the guest boots on a fresh kernel.
Reach for it when a running sandbox stops answering commands.
A sandbox that is not running answers `409 SANDBOX_NOT_RUNNING`: resume it instead.

> **Note: Related**
>
> [Timeouts and auto-stop](/docs/develop/sandboxes/timeouts) covers what happens when `timeout_sec` elapses, [Snapshots and persistence](/docs/develop/snapshots) covers snapshots and forks, and [Sizing and limits](/docs/develop/sandboxes/sizing) covers what each state costs.
