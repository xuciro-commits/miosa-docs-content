---
slug: "develop/sandboxes/create"
title: "Create and connect"
description: "Create a sandbox with the fused run endpoint or a full create, manage its lifetime in Python context managers, and connect to an existing sandbox by id or name."
section: "Sandboxes"
order: 2002
---

Two ways to get a running sandbox: the fused **run** endpoint when you want a sandbox and an immediate first command, and a full **create** when you need to provision first, write files, start a preview server, or keep the ID for a longer session.

## Fast path: create and run

For agents that need a sandbox and immediately run the first command, use the fused run endpoint. It is faster because MIOSA owns the readiness wait on the server and avoids an extra client poll loop plus a second public exec request.

**Python**

```python
import os
import requests

response = requests.post(
    "https://api.miosa.ai/api/v1/sandboxes/run",
    headers={
        "Authorization": f"Bearer {os.environ['MIOSA_API_KEY']}",
        "Content-Type": "application/json",
        "Idempotency-Key": "run-2026-06-09-001",
    },
    json={
        "template_id": "miosa-sandbox",
        "command": "python3 -c \"print('ready')\"",
        "size": "small",
        "timeout_sec": 300,
        "wait_timeout_ms": 30000,
        "env": {"NODE_ENV": "production"},
        "metadata": {"agent_run": "abc123"},
    },
)
response.raise_for_status()

run = response.json()
print(run["data"]["id"], run["data"]["state"])
print(run["exec"]["stdout"])
print(run["timings"]["server_wait_and_exec_ms"])
```

**TypeScript**

```typescript
const response = await fetch("https://api.miosa.ai/api/v1/sandboxes/run", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.MIOSA_API_KEY!}`,
    "Content-Type": "application/json",
    "Idempotency-Key": "run-2026-06-09-001"
  },
  body: JSON.stringify({
    template_id: "miosa-sandbox",
    command: "node -e \"console.log('ready')\"",
    size: "small",
    timeout_sec: 300,
    wait_timeout_ms: 30000,
    env: { NODE_ENV: "production" },
    metadata: { agent_run: "abc123" }
  })
});

if (!response.ok) throw new Error(await response.text());

const run = await response.json();
console.log(run.data.id, run.data.state);
console.log(run.exec.stdout);
console.log(run.timings.server_wait_and_exec_ms);
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/run \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: run-2026-06-09-001" \
  -d '{
    "template_id": "miosa-sandbox",
    "command": "python3 -c \"print(1+1)\"",
    "size": "small",
    "timeout_sec": 300,
    "wait_timeout_ms": 30000,
    "metadata": { "agent_run": "abc123" }
  }'
```

Response shape:

```json
{
  "data": {
    "id": "sbx_a1b2c3d4",
    "state": "running",
    "template_id": "miosa-sandbox",
    "ready": true
  },
  "exec": {
    "stdout": "2\n",
    "stderr": "",
    "exit_code": 0
  },
  "timings": {
    "server_wait_and_exec_ms": 512
  }
}
```

## Create with full options

Use full create when you need to provision the sandbox first, write files, start a preview server, attach databases, or keep the ID for a longer session.

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sbx = miosa.sandboxes.create(
    "miosa-sandbox",              # template_id (positional); default if omitted
    cpu_count=2,
    memory_mb=4096,
    disk_size_mb=10240,
    timeout_sec=3600,             # idle-anchored; resets on activity; default 3600
    env={"NODE_ENV": "production"},
    metadata={"agent_run": "abc123"},
    external_workspace_id="acme-corp",   # white-label attribution
    external_user_id="user_123",
    external_project_id="project_456",
    idempotency_key="run-2026-06-09-001",
)

print(sbx.id)
print(sbx.state)
print(sbx.ready)
```

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const sbx = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  cpuCount: 2,
  memoryMb: 4096,
  diskSizeMb: 10240,
  timeoutSec: 3600,
  env: { NODE_ENV: "production" },
  metadata: { agentRun: "abc123" },
  externalWorkspaceId: "acme-corp",
  externalUserId: "user_123",
  externalProjectId: "project_456",
  idempotencyKey: "run-2026-06-09-001",
});

console.log(sbx.id, sbx.state, sbx.boot_ms);
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: run-2026-06-09-001" \
  -d '{
    "template_id": "miosa-sandbox",
    "cpu_count": 2,
    "memory_mb": 4096,
    "disk_size_mb": 10240,
    "timeout_sec": 3600,
    "env": { "NODE_ENV": "production" },
    "metadata": { "agent_run": "abc123" },
    "external_workspace_id": "acme-corp",
    "external_user_id": "user_123",
    "external_project_id": "project_456"
  }'
```

The response body contains `id`, `state`, `template_id`, `size`, `resource_contract`,
`cpu_count`, `memory_mb`, and `created_at`. Save the `id`; every
subsequent call needs it.

### Context manager (auto-destroy)

The Python SDK supports `with` / `async with`; the sandbox is destroyed on exit
even if an exception is raised:

**Python**

```python
with miosa.sandboxes.create("miosa-sandbox") as sbx:
    sbx.files.write("/workspace/hello.py", "print('hello')")
    result = sbx.exec.run("python3 /workspace/hello.py")
    print(result.stdout)
# sbx.destroy() called automatically here
```

**TypeScript**

```typescript
// TypeScript: manual cleanup with try/finally
const sbx = await miosa.sandboxes.create({ templateId: "miosa-sandbox" });
try {
  await sbx.files.write("/workspace/hello.py", "print('hello')");
  const result = await sbx.exec.run("python3 /workspace/hello.py");
  console.log(result.stdout);
} finally {
  await sbx.destroy();
}
```

### Connect to an existing sandbox

**Python**

```python
# By ID
sbx = miosa.sandboxes.get("sbx_a1b2c3d4")

# By name (if you set name= at create time)
sbx = miosa.sandboxes.get_by_name("my-dev-env")

# connect() is an alias for get()
sbx = miosa.sandboxes.connect("sbx_a1b2c3d4")
```

**TypeScript**

```typescript
const sbx = await miosa.sandboxes.get("sbx_a1b2c3d4");
// or
const sbx = await miosa.sandboxes.getByName("my-dev-env");
```

**cURL**

```bash
curl https://api.miosa.ai/api/v1/sandboxes/sbx_a1b2c3d4 \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# By name
curl https://api.miosa.ai/api/v1/sandboxes/by-name/my-dev-env \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## List and filter sandboxes

**Python**

```python
# All running sandboxes
running = miosa.sandboxes.list(status="running")  # status: "running" | "paused" | "destroyed"

# Filter by name (prefix match)
named = miosa.sandboxes.list(name="my-dev")

# Filter by white-label workspace
ws_sandboxes = miosa.sandboxes.list(external_workspace_id="acme-corp")

# Filter by tags
tagged = miosa.sandboxes.list(tags=["pr-preview", "staging"])
```

**TypeScript**

```typescript
const running = await miosa.sandboxes.list({ status: "running" });  // status: "running" | "paused" | "destroyed"
const named   = await miosa.sandboxes.list({ name: "my-dev" });
const byCorp  = await miosa.sandboxes.list({ externalWorkspaceId: "acme-corp" });
```

**cURL**

```bash
curl "https://api.miosa.ai/api/v1/sandboxes?status=running" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl "https://api.miosa.ai/api/v1/sandboxes?name=my-dev" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

> **Note: Related**
>
> See [Lifecycle](/docs/develop/sandboxes/lifecycle) for the state machine these calls move through, [Environment variables](/docs/develop/sandboxes/environments) for create-time and exec-level env, and [Sizing and limits](/docs/develop/sandboxes/sizing) for the resource contracts referenced above.
