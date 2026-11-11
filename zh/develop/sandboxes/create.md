---
slug: "develop/sandboxes/create"
title: "创建并连接"
description: "使用融合运行端点或完整创建创建沙箱，在 Python 上下文管理器中管理其生命周期，并通过 ID 或名称连接到现有沙箱。"
section: "Sandboxes"
order: 2002
---

获取正在运行的沙箱的两种方法：当您需要沙箱和立即执行的第一个命令时，使用融合的 **运行** 端点；当您需要首先配置、写入文件、启动预览服务器或保留 ID 以进行较长的会话时，使用完整的 **创建**。

## 快速路径：创建并运行

对于需要沙箱并立即运行第一个命令的智能体，请使用融合运行端点。它更快，因为 MIOSA 拥有服务器上的就绪等待，并避免了额外的客户端轮询循环和第二个公共执行请求。

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

**打字稿**

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

**curl**

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

响应形状：

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

## 使用完整选项创建

当您需要首先配置沙箱、写入文件、启动预览服务器、附加数据库或保留 ID 以进行较长会话时，请使用完整创建。

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

**打字稿**

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

**curl**

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

响应正文包含 `id`、`state`、`template_id`、`size`、`resource_contract`、
`cpu_count`、`memory_mb` 和 `created_at`。保存`id`；每个
后续调用需要它。

### 上下文管理器（自动销毁）

Python SDK 支持 `with` / `async with`；沙箱在退出时被破坏
即使引发异常：

**Python**

```python
with miosa.sandboxes.create("miosa-sandbox") as sbx:
    sbx.files.write("/workspace/hello.py", "print('hello')")
    result = sbx.exec.run("python3 /workspace/hello.py")
    print(result.stdout)
# sbx.destroy() called automatically here
```

**打字稿**

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

### 连接到现有沙箱

**Python**

```python
# By ID
sbx = miosa.sandboxes.get("sbx_a1b2c3d4")

# By name (if you set name= at create time)
sbx = miosa.sandboxes.get_by_name("my-dev-env")

# connect() is an alias for get()
sbx = miosa.sandboxes.connect("sbx_a1b2c3d4")
```

**打字稿**

```typescript
const sbx = await miosa.sandboxes.get("sbx_a1b2c3d4");
// or
const sbx = await miosa.sandboxes.getByName("my-dev-env");
```

**curl**

```bash
curl https://api.miosa.ai/api/v1/sandboxes/sbx_a1b2c3d4 \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# By name
curl https://api.miosa.ai/api/v1/sandboxes/by-name/my-dev-env \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## 列出并过滤沙箱

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

**打字稿**

```typescript
const running = await miosa.sandboxes.list({ status: "running" });  // status: "running" | "paused" | "destroyed"
const named   = await miosa.sandboxes.list({ name: "my-dev" });
const byCorp  = await miosa.sandboxes.list({ externalWorkspaceId: "acme-corp" });
```

**curl**

```bash
curl "https://api.miosa.ai/api/v1/sandboxes?status=running" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl "https://api.miosa.ai/api/v1/sandboxes?name=my-dev" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

> **注意：相关**
>
> 请参阅[生命周期](/docs/develop/sandboxes/lifecycle)了解这些调用所经过的状态机，[环境变量](/docs/develop/sandboxes/environments)了解创建时和执行级环境，以及[大小调整和限制](/docs/develop/sandboxes/sizing)了解上面引用的资源契约。
