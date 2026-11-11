---
slug: "develop/sandboxes/long-running-tasks"
title: "Long-running tasks"
description: "Work that outlives a single exec: background processes, their logs and status, always-on sandboxes, and what comes back after pause and resume."
section: "Sandboxes"
order: 2011
---

A single `exec` call is bounded: the sandbox `exec` endpoint accepts a `timeout` of 1 to 86,400 seconds (default 3,600), independent of the sandbox's own timeout. Work that runs longer - a dev server, a build, a watcher, a worker - should be started as a background process and inspected separately.

## Start a background process

**Python**

```python
proc = client.sandboxes.processes.start(
    "sbx_abc123",
    cmd="npm run dev -- --port 3000",
    cwd="/workspace",
    name="dev-server",
)
print(proc.pid)   # proc_...
```

**TypeScript**

```typescript
const proc = await client.sandboxes.processes.start('sbx_abc123', {
  cmd: 'npm run dev -- --port 3000',
  cwd: '/workspace',
  name: 'dev-server',
});
```

**CLI**

```bash
miosa process start --name build --cwd /workspace/app -- npm run build
miosa ps            # list processes (alias for: miosa process list)
```

The body accepts `cmd` (required), `cwd`, `env`, and `name`. The sandbox must be `running` to start a process.

## Check status and logs

| Status | Meaning |
|---|---|
| `running` | Process is alive |
| `exited` | Process exited normally |
| `killed` | Process was stopped via `DELETE` |
| `error` | Process failed to start |

Fetch the last N lines of stdout+stderr, or stream them live:

```bash
# Last 50 lines
curl "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/processes/proc_01hwxyz/logs?tail=50" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Live stream (SSE)
curl "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/processes/proc_01hwxyz/stream" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Stop a process with `DELETE`, which sends `SIGTERM` and then `SIGKILL` after a grace period.

## Daemons for long-lived services

A sandbox daemon is a named long-lived process: start it once, then list it, read
its output and restart or stop it by name instead of hunting for a pid.

```bash
miosa daemon create my-box --name web --command "npm run dev" --cwd /workspace/app
miosa daemon list my-box
miosa daemon logs my-box web
miosa daemon restart my-box web
miosa daemon stop my-box web
```

The same calls are `POST /api/v1/sandboxes/:id/daemons`, `GET .../daemons` and
`GET .../daemons/:name`; the [Processes API](/docs/api-reference/sandbox-processes)
documents the process model they sit on.

On Computers, long-lived daemons belong to systemd and restart on failure rather
than to a process table; see the [Services API](/docs/api-reference/services).

## Keep a sandbox alive

A background process only runs while the sandbox does. Two settings control that:

- **`always_on: true`** disables timeout and idle-timeout enforcement until an explicit lifecycle action, subject to tenant policy.
- **`idle_timeout_sec`** stops an abandoned persistent session after a period of inactivity; `0` disables idle auto-stop. Left unset, it defaults to `0`, or `1800` (30 minutes) for accounts created on or after October 11, 2026. A stopped sandbox wakes on the next request.

The timeout is idle-anchored.
For persistent sandboxes, expiry stops the sandbox rather than destroying it, and a stop never keeps running processes.
Explicitly non-persistent sandboxes are destroyed. See [Timeouts and auto-stop](/docs/develop/sandboxes/timeouts).

> **Caution: Processes do not survive a stop, snapshot, or fork**
>
> A background process lives for the lifetime of its running session. It never survives a stop, and it is not preserved across a snapshot, restore, or fork. Only a pause (Firecracker only) keeps processes, and only for up to 30 minutes, after which the sandbox behaves like a stop. Re-start the process after the sandbox is running again, or make it a declared service so it is easy to relaunch.

## See also

- [Processes API](/docs/api-reference/sandbox-processes): Endpoints, request fields, and status values.
- [Services API](/docs/api-reference/services): systemd-backed services on a Computer.
- [Timeouts and auto-stop](/docs/develop/sandboxes/timeouts): The active-session timeout and how usage is measured.
