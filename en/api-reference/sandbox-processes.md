---
slug: "api-reference/sandbox-processes"
title: "Sandbox Processes API"
description: "Start, monitor, and stream logs for long-running background processes inside sandboxes."
section: "API reference"
order: 12023
---

The processes API lets you launch long-running background commands inside a sandbox and inspect their output  -  useful for starting dev servers, watchers, or any process that runs beyond a single exec call.

Base path: `/api/v1/sandboxes/{id}/processes`

> **Note**
>
> Processes are managed by the sandbox runtime and survive for the lifetime of the sandbox. They are not preserved across snapshots or forks.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/sandboxes/{id}/processes` | Start a process |
| `GET` | `/api/v1/sandboxes/{id}/processes` | List processes |
| `GET` | `/api/v1/sandboxes/{id}/processes/{pid}` | Get process status |
| `DELETE` | `/api/v1/sandboxes/{id}/processes/{pid}` | Stop a process |
| `GET` | `/api/v1/sandboxes/{id}/processes/{pid}/logs` | Fetch recent log lines |
| `GET` | `/api/v1/sandboxes/{id}/processes/{pid}/stream` | SSE live log stream |

---

## Start a Process

**`POST /api/v1/sandboxes/{id}/processes`**

### Auth

```http
Authorization: Bearer msk_...
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cmd` | string | Yes | Command to run (passed to `/bin/sh -c`) |
| `cwd` | string | No | Working directory (default: `/workspace`) |
| `env` | object | No | Additional environment variables |
| `name` | string | No | Human-readable label |

```json
{
  "cmd": "npm run dev -- --port 3000",
  "cwd": "/workspace",
  "name": "dev-server"
}
```

### Response  -  `201 Created`

```json
{
  "data": {
    "pid": "proc_01hwxyz...",
    "name": "dev-server",
    "cmd": "npm run dev -- --port 3000",
    "cwd": "/workspace",
    "status": "running",
    "started_at": "2026-05-26T10:14:23Z",
    "exited_at": null,
    "exit_code": null
  }
}
```

---

## List Processes

**`GET /api/v1/sandboxes/{id}/processes`**

### Response  -  `200 OK`

```json
{
  "data": [
    {
      "pid": "proc_01hwxyz...",
      "name": "dev-server",
      "status": "running",
      "started_at": "2026-05-26T10:14:23Z",
      "exited_at": null,
      "exit_code": null
    }
  ]
}
```

---

## Get Process Status

**`GET /api/v1/sandboxes/{id}/processes/{pid}`**

### Response  -  `200 OK`

```json
{
  "data": {
    "pid": "proc_01hwxyz...",
    "name": "dev-server",
    "cmd": "npm run dev -- --port 3000",
    "cwd": "/workspace",
    "status": "exited",
    "started_at": "2026-05-26T10:14:23Z",
    "exited_at": "2026-05-26T10:22:01Z",
    "exit_code": 0
  }
}
```

**Process status values:**

| Status | Description |
|--------|-------------|
| `running` | Process is alive |
| `exited` | Process exited normally |
| `killed` | Process was stopped via DELETE |
| `error` | Process failed to start |

---

## Stop a Process

**`DELETE /api/v1/sandboxes/{id}/processes/{pid}`**

Sends `SIGTERM` followed by `SIGKILL` after a grace period.

### Response  -  `204 No Content`

---

## Fetch Logs

**`GET /api/v1/sandboxes/{id}/processes/{pid}/logs`**

Returns the last N lines of stdout+stderr.

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `tail` | integer | `200` | Number of lines to return |

### Response  -  `200 OK`

```json
{
  "data": {
    "lines": [
      "  > Local:   http://localhost:3000",
      "  > Network: http://0.0.0.0:3000",
      "  ready in 743ms"
    ]
  }
}
```

---

## Live Log Stream (SSE)

**`GET /api/v1/sandboxes/{id}/processes/{pid}/stream`**

Streams stdout+stderr as SSE. Uses `text/event-stream`  -  requires Bearer token auth.

```text
id: log_001
event: log
data: {"line":"  ready in 743ms","ts":"2026-05-26T10:14:24Z"}

: heartbeat
```

---

## Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `INVALID_ID` | Sandbox ID is not a valid UUID |
| 404 | `NOT_FOUND` | Sandbox or process does not exist |
| 409 | `SANDBOX_NOT_RUNNING` | Sandbox must be in `running` state to start processes |
| 500 | `PROCESS_START_FAILED` | Runtime failed to spawn the process |

---

## Examples

**cURL**

```bash
# Start
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/processes \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"cmd": "npm run dev -- --port 3000", "name": "dev-server"}'

# List
curl https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/processes \
  -H "Authorization: Bearer msk_u_..."

# Logs
curl "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/processes/proc_01hwxyz/logs?tail=50" \
  -H "Authorization: Bearer msk_u_..."

# Stop
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/processes/proc_01hwxyz \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa, time

client = miosa.Miosa()

# Start a dev server
proc = client.sandboxes.processes.start(
    "sbx_abc123",
    cmd="npm run dev -- --port 3000",
    name="dev-server",
)

# Wait a moment then check logs
time.sleep(3)
logs = client.sandboxes.processes.logs("sbx_abc123", proc.pid, tail=50)
for line in logs:
    print(line)

# Stop
client.sandboxes.processes.stop("sbx_abc123", proc.pid)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Start
const proc = await client.sandboxes.processes.start('sbx_abc123', {
  cmd: 'npm run dev -- --port 3000',
  name: 'dev-server',
});

// Poll status
let status = proc.status;
while (status === 'running') {
  await new Promise(r => setTimeout(r, 2000));
  const updated = await client.sandboxes.processes.get('sbx_abc123', proc.pid);
  status = updated.status;
}

// Logs
const { data } = await client.sandboxes.processes.logs('sbx_abc123', proc.pid);
console.log(data.lines.join('\n'));
```
