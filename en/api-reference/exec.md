---
slug: "api-reference/exec"
title: "Exec API"
description: "API reference for executing bash commands and Python code on MIOSA computers."
section: "API reference"
order: 12018
---

Run commands and scripts directly on a computer without desktop interaction. Both endpoints block until the command exits and return combined stdout+stderr.

Base path: `/api/v1/computers/{id}/exec`

> **Caution**
>
> The computer must be in `"running"` status. Commands run as the default user inside the VM. The bash exec timeout is capped at 86,400 s; the Python exec timeout is capped at 300 s.

> **Caution**
>
> Rate limits vary by key, tenant policy, and endpoint family. Honor `X-RateLimit-*` and `Retry-After` response headers. For long-running output, prefer the streaming exec endpoint which avoids holding an HTTP connection open for the full duration.

---

## Execute Bash Command

**`POST /api/v1/computers/{id}/exec`**

Runs a shell command on the computer.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `command` | string | Yes | Shell command to execute |
| `timeout` | integer | No | Timeout in seconds (default: 30, max: 86400) |

### Response  -  `200 OK`
```json
{
  "output": "total 24\ndrwxr-xr-x 6 user user 4096 Apr 11 10:00 .\n",
  "exit_code": 0
}
```

The `output` field contains combined stdout and stderr.

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 409 | `COMPUTER_NOT_RUNNING` | Computer is not running |
| 502 |  -  | In-VM agent unreachable |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "ls -la /workspace", "timeout": 30}'
```

### Examples
```bash
# Install a package
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "sudo apt-get install -y jq", "timeout": 120}'

# Check disk space
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "df -h"}'

# Multi-line script
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "cd /workspace && mkdir -p project && echo done"}'
```

---

## Execute Python Code

**`POST /api/v1/computers/{id}/exec/python`**

Runs Python code on the computer. The code is written to a temporary file and executed with `python3`.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | string | Yes | Python source code |
| `timeout` | integer | No | Timeout in seconds (default: 30, max: 300) |

### Response  -  `200 OK`
```json
{
  "output": "42\n",
  "exit_code": 0
}
```

### How It Works

1. Code is written to `/tmp/miosa_exec_&lt;random&gt;.py`
2. Executed via `timeout &lt;N&gt; python3 /tmp/miosa_exec_&lt;random&gt;.py 2&gt;&1`
3. Temporary file is deleted after execution
4. stdout and stderr are captured in `output`

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 409 | `COMPUTER_NOT_RUNNING` | Computer is not running |
| 502 |  -  | In-VM agent unreachable |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/python \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "import sys\nprint(f\"Python {sys.version}\")\nprint(2 + 2)",
    "timeout": 30
  }'
```

### Examples
```bash
# JSON processing
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/python \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "import json\ndata = {\"hello\": \"world\"}\nprint(json.dumps(data, indent=2))",
    "timeout": 10
  }'

# File processing
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/python \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "import os\nfor f in os.listdir(\"/workspace\"):\n    print(f)",
    "timeout": 10
  }'

# Long-running computation
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/python \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "total = sum(range(10_000_000))\nprint(f\"Sum: {total}\")",
    "timeout": 60
  }'
```

---

## Terminal

### Create Terminal Session

**`POST /api/v1/computers/{id}/terminal`**

Creates a WebSocket-based terminal session on the computer.

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cols` | integer | No | Terminal columns (default: 80) |
| `rows` | integer | No | Terminal rows (default: 24) |
| `shell` | string | No | Shell to use (default: `/bin/bash`) |

#### Response  -  `201 Created`
```json
{
  "data": {
    "session_id": "default",
    "ws_url": "wss://my-computer.sandbox.miosa.ai/ws/terminal/550e8400-.../default?auth=stream_token",
    "stream_auth": "stream_token",
    "expires_at": 1712700060,
    "computer_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/terminal \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"cols": 80, "rows": 24, "shell": "/bin/bash"}'
```

---

### Resize Terminal

**`POST /api/v1/computers/{id}/pty/{session_id}/resize`**

Resizes an active terminal session.

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cols` | integer | Yes | New column count |
| `rows` | integer | Yes | New row count |

#### Response  -  `200 OK`
```json
{
  "status": "ok"
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/pty/{session_id}/resize \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"cols": 120, "rows": 40}'
```

---

## Timeout Behavior

- Commands exceeding the timeout are killed with SIGTERM
- The bash exec timeout is capped at **86,400 seconds**
- The Python exec timeout is clamped: `min(requested, 300)`
- If no timeout is specified, the default is **30 seconds**

## Pre-installed Software

The default template includes:

- Python 3 with pip
- Node.js (if installed via selected_apps)
- Common CLI tools: curl, wget, git, vim, jq
- System utilities: htop, tree, zip/unzip

Install additional packages via the bash exec endpoint:
```bash
# Python packages
{"command": "pip install requests pandas numpy"}

# System packages
{"command": "sudo apt-get install -y postgresql-client"}
```

---

---

## Sandbox exec

The same exec surface exists for sandboxes under `/api/v1/sandboxes/{id}`.

> **Caution: SOMA command size**
>
> On a SOMA sandbox the executable and each argument are limited to 4096 bytes, and a longer shell command answers `502 AGENT_UNAVAILABLE`, which looks retryable but is not.
> Send the command as a file, or on standard input, instead.
> See [Choosing a runtime](/docs/soma/choosing-a-runtime).

### Execute a Command (sandbox)

**`POST /api/v1/sandboxes/{id}/exec`**

Runs a shell command inside the sandbox. Blocks until the process exits.

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Sandbox ID. Must be in `running` state. |

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `command` | string | Yes | Shell command to execute. |
| `timeout` | integer | No | Per-command timeout in seconds. Default `3600`, max `86400`. |
| `cwd` | string | No | Working directory inside the VM. |
| `env` | object | No | Env vars for this invocation only. |

#### Response  -  `200 OK`
```json
{
  "data": {
    "sandbox_id": "sbx_01j9xr2t4fk8me3n5q",
    "stdout": "2\n",
    "stderr": "",
    "exit_code": 0
  }
}
```

#### Errors

| Status | Code | Cause |
|--------|------|-------|
| 409 | `SANDBOX_NOT_RUNNING` | Sandbox is not in `running` state. |
| 504 |  -  | Command timed out. |
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "python3 -c \"print(1+1)\"", "timeout": 30}'
```

### Open a Terminal Session (sandbox)

**`POST /api/v1/sandboxes/{id}/terminal`**

Creates a PTY session inside a running sandbox and returns a short-lived stream
token for browser WebSocket clients.

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cols` | integer | No | Initial terminal width. Defaults to `80`. |
| `rows` | integer | No | Initial terminal height. Defaults to `24`. |
| `shell` | string | No | Shell command. Defaults to login bash when available. |

#### Response  -  `201 Created`
```json
{
  "session_id": "52d1df5a0a3a0018",
  "ws_url": "wss://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/terminal/stream",
  "stream_auth": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9...",
  "stream_auth_expires_at": "2026-05-13T12:30:00Z"
}
```

Open the WebSocket with:
```text
wss://api.miosa.ai/api/v1/sandboxes/{id}/terminal/stream?session_id={session_id}&cols=120&rows=32&token={stream_auth}
```

Clients should send raw terminal input as binary frames. Resize events are sent
as JSON text frames:
```json
{"type":"resize","cols":120,"rows":32}
```

Delete the session when finished:
```bash
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/{id}/terminal/{session_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## See also

- [Streaming Exec](/docs/api-reference/streaming-exec)  -  SSE stream for long-running commands
- [Computers API](/docs/api-reference/computers)  -  computer CRUD and lifecycle
- [Files API](/docs/api-reference/files)  -  read and write files without exec
- [Error Codes](/docs/api-reference/errors)  -  `COMPUTER_NOT_RUNNING`, `AGENT_UNAVAILABLE`
