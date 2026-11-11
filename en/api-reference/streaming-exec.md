---
slug: "api-reference/streaming-exec"
title: "Streaming Exec API"
description: "API reference for streaming command output in real-time from MIOSA computers via SSE."
section: "API reference"
order: 12019
---

Streaming exec runs a command on a computer and delivers output line-by-line via Server-Sent Events. Use it for long-running builds, installs, or any command where you want incremental output rather than waiting for completion.

Base path: `/api/v1/computers/{id}/exec/stream`

> **Note**
>
> Standard `POST /computers/{id}/exec` waits for the command to finish before returning. Streaming exec is better for commands that run longer than a few seconds or that you want to monitor interactively.

---

## Quick Start
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Stream a long-running build
for await (const event of client.exec.stream(computerId, {
  command: 'pip install torch',
  timeout: 300,
})) {
  if (event.type === 'output') {
    process.stdout.write(event.data.line);
  } else if (event.type === 'exit') {
    console.log(`Exit code: ${event.data.exitCode}`);
    break;
  }
}
```
```bash
# Obtain a streaming ticket first, then connect
TICKET=$(curl -s -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/stream \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "pip install torch", "timeout": 300}' | jq -r .ticket)

curl -N "https://api.miosa.ai/api/v1/computers/{id}/exec/stream/events?ticket=$TICKET" \
  -H "Accept: text/event-stream"
```

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/computers/{id}/exec/stream` | Start a streaming exec and obtain a ticket |
| `GET` | `/computers/{id}/exec/stream/events` | SSE stream (requires ticket) |
| `POST` | `/computers/{id}/exec` | Non-streaming exec (waits for completion) |
| `POST` | `/computers/{id}/exec/python` | Non-streaming Python exec |

---

## Start Streaming Exec

**`POST /api/v1/computers/{id}/exec/stream`**

Starts the command and returns a short-lived ticket for the SSE stream.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `command` | string | Yes | Shell command to execute |
| `timeout` | integer | No | Timeout in seconds (default: 30, max: 86400) |
| `shell` | string | No | Shell to use (default: `/bin/bash`) |

### Response  -  `202 Accepted`
```json
{
  "ticket": "exec_short_lived_token",
  "expires_at": 1712700060,
  "stream_url": "/api/v1/computers/{id}/exec/stream/events?ticket=exec_short_lived_token"
}
```

The ticket expires in 60 seconds  -  open the EventSource connection immediately.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 409 | `COMPUTER_NOT_RUNNING` | Computer is not running |
| 502 | `AGENT_UNAVAILABLE` | In-VM agent unreachable |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/stream \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "npm install", "timeout": 120}'
```

---

## Connect to SSE Stream

**`GET /api/v1/computers/{id}/exec/stream/events?ticket={ticket}`**

### Event Types

| Event | Payload Fields | Description |
|-------|---------------|-------------|
| `output` | `line`, `stream` | A line of stdout or stderr |
| `exit` | `exit_code`, `duration_ms` | Command finished; stream closes after this |
| `timeout` | `timeout_seconds` | Command killed after timeout |
| `error` | `message` | Internal error before command started |

### Event Payload Examples
```text

event: output
data: {"line": "Collecting torch\n", "stream": "stdout"}

event: output
data: {"line": "  Downloading torch-2.3.0-...\n", "stream": "stdout"}

event: exit
data: {"exit_code": 0, "duration_ms": 45312}
```

### `stream` Values

| Value | Description |
|-------|-------------|
| `stdout` | Standard output |
| `stderr` | Standard error |
```javascript
// Browser
const es = new EventSource(`/api/v1/computers/${id}/exec/stream/events?ticket=${ticket}`);

es.addEventListener('output', e => {
  const { line, stream } = JSON.parse(e.data);
  console.log(`[${stream}] ${line}`);
});

es.addEventListener('exit', e => {
  const { exit_code } = JSON.parse(e.data);
  console.log(`Exited with code ${exit_code}`);
  es.close();
});

es.addEventListener('error', () => {
  // Connection dropped or ticket expired
  es.close();
});
```

---

## Comparison: Streaming vs Non-Streaming

| Aspect | `POST /exec` | `POST /exec/stream` |
|--------|-------------|---------------------|
| Response timing | After command completes | Immediate (ticket) |
| Output delivery | All at once | Line by line via SSE |
| Max timeout | 86400 s | 86400 s |
| Best for | Quick commands | Long builds, installs |
| SDK method | `exec.run()` | `exec.stream()` |

---

## Common Recipes

### Monitor a build
```python
for event in client.exec.stream(computer_id, command="make build", timeout=300):
    if event.type == "output":
        print(event.data["line"], end="")
    elif event.type == "exit":
        if event.data["exit_code"] != 0:
            raise RuntimeError(f"Build failed with exit code {event.data['exit_code']}")
        print("Build succeeded")
        break
```

### Capture both streams separately
```typescript
const stdout: string[] = [];
const stderr: string[] = [];

for await (const event of client.exec.stream(computerId, { command: 'myapp 2>&1 | tee /tmp/output.log' })) {
  if (event.type === 'output') {
    if (event.data.stream === 'stdout') stdout.push(event.data.line);
    else stderr.push(event.data.line);
  } else if (event.type === 'exit') {
    break;
  }
}
```

### Timeout handling
```python
try:
    for event in client.exec.stream(computer_id, command="long-task", timeout=60):
        if event.type == "timeout":
            print(f"Command killed after {event.data['timeout_seconds']}s")
            break
        elif event.type == "exit":
            break
except Exception as e:
    print(f"Stream error: {e}")
```

---

## Sandbox streaming exec

Sandboxes stream command output directly over SSE, without a separate ticket step.

**`POST /api/v1/sandboxes/{id}/exec/stream`**

Runs a command and streams stdout/stderr events as Server-Sent Events. Use this for installer output, long-running agent tasks, and build logs that should show progress before the command exits. Request body is the same as `/exec`.

```bash
curl -N -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/exec/stream \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Accept: text/event-stream" \
  -H "Content-Type: application/json" \
  -d '{"command":"npm install && npm run build","working_dir":"/workspace","timeout":300}'
```

Example events:
```text
event: stdout
data: {"line":"added 342 packages"}

event: exit
data: {"exit_code":0}
```
