---
slug: "api-reference/open-computers"
title: "OpenComputersAPI"
description: "通过 MIOSA API - exec、文件、隧道、AI 智能体、推理集群和机密，注册和控制客户拥有的计算机，包括本地服务器。"
section: "API reference"
order: 12031
---

OpenComputers 允许您注册已拥有的物理机或虚拟机（Mac、Linux、Windows）并通过 MIOSA 的 API 控制它们 - 运行命令、管理文件、公开 HTTP 隧道、调度 AI 智能体、构建推理集群和管理机密。

主机可以是本地服务器、工作站或托管提供商的现有计算机。
可用的操作取决于安装的智能体和主机的功能。
有关硬件、连接和数据责任，请参阅 [OpenComputers](/docs/opencomputers)。

基本路径：`/api/v1/opencomputers`

支持的动词：**GET**（列表/显示）、**POST**（注册/创建/执行/调度）、**PATCH**（更新标签/隧道）、**DELETE**（撤销/取消）。

> **注意**
>
> 速率限制因密钥、租户策略和端点系列而异。尊重 `X-RateLimit-*` 和 `Retry-After` 响应标头。 Exec 作业和智能体会话异步运行 - 轮询或流式传输结果而不是保持连接。

> **注意**
>
> 注册主机并按照生成的说明连接目标计算机上受支持的 OpenComputers 智能体。准确显示注册时返回的 **主机密钥**
> 一次 - 安全存放。

---

## 主持人

### 列出主机

**`GET /api/v1/opencomputers/hosts`**

返回带有 `hosts` 数组的对象。
选择每个主机之前，请检查其 `id`、`state`、`capabilities` 和 `last_heartbeat`。
准确使用返回的ID；下面的示例使用占位符。

```json
{
  "hosts": [
    {
      "id": "<host-id>",
      "name": "lab-server",
      "state": "active",
      "capabilities": ["exec"],
      "last_heartbeat": "2026-09-08T12:00:00Z"
    }
  ]
}
```

这是一个简短的回复。
最近的心跳和已完成的操作比单独的主机记录更能证明准备就绪。

---

### 注册一个主机

**`POST /api/v1/opencomputers/hosts`**

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `name` |字符串|是的 |主机的显示名称 |
| `region` |字符串|没有 |区域标签|
| `platform` |字符串|没有 |主机操作系统选择|

注册返回`{ host, host_key, control_url, warning }`。
`host`对象包含新的主机ID； `host_key` 对已安装的智能体进行身份验证。
使用您的平台 API 密钥进行 REST 请求，而不是主机密钥。

> **注意**
>
> `host_key` 在注册时**仅返回一次**。主机智能体使用
> 它进行身份验证。立即存储它 - 以后无法检索。

---

### 获取主机

**`GET /api/v1/opencomputers/hosts/{id}`**

---

### 撤销主机

**`DELETE /api/v1/opencomputers/hosts/{id}`**

返回 `204 No Content`。主机智能体将无法再进行身份验证。

---

### 主机事件流（SSE）

**`GET /api/v1/opencomputers/hosts/{id}/events`**

将来自主机的实时生命周期事件作为服务器发送的事件进行流式传输。
```text

event: status_change
data: {"type":"status_change","host_id":"host_abc","data":{"status":"online"},"timestamp":"..."}
```

---

## 职位

在注册的主机上运行 shell 命令并检索输出。

### 运行作业

**`POST /api/v1/opencomputers/hosts/{id}/exec`**

|领域|类型 |必填 |描述 |
|---|---|---|---|
| `cmd` |字符串|是的 |执行命令 |
| `args` |字符串[] |没有 |争论分别通过|
| `env` |字符串[] |没有 | `KEY=VALUE` 格式的环境条目 |
| `cwd` |字符串|没有 |工作目录 |
| `timeout_ms` |整数 |没有 |超时时间（以毫秒为单位）；默认 30000 |
| `stream` |布尔 |没有 |默认`true`：流SSE； `false`：收集 JSON 格式的输出 |

对于 `stream: true`，POST 响应携带 `exec_chunk`、`exec_result` 或 `exec_error` 事件。
保持连接打开，直到出现最终结果。
使用 `stream: false`，完整的命令返回：

```json
{
  "job_id": "<job-id>",
  "exit_code": 0,
  "stdout": "lab-server\n",
  "stderr": ""
}
```

超时或断开连接的客户端并不能证明进程已停止。
在重试具有副作用的操作之前检查作业和主机。

---

### 列出职位

**`GET /api/v1/opencomputers/hosts/{id}/jobs`**

---

### 找工作

**`GET /api/v1/opencomputers/hosts/{id}/exec/{job_id}`**

---

### 流作业输出 (SSE)

**`GET /api/v1/opencomputers/hosts/{id}/exec/{job_id}/stream`**

作业运行时实时传输 stdout/stderr。

---

### 取消工作

**`DELETE /api/v1/opencomputers/hosts/{id}/exec/{job_id}`**

返回 `204 No Content`。

---

## 文件系统

管理已注册主机上的文件和目录。

| 端点 | 方法 | 描述 |
|----------|--------|-------------|
| `/opencomputers/hosts/{id}/fs/list` | GET |列出目录 (`?path=`) |
| `/opencomputers/hosts/{id}/fs/stat` | GET |统计路径 (`?path=`) |
| `/opencomputers/hosts/{id}/fs/download` | GET |下载文件 (`?path=`) |
| `/opencomputers/hosts/{id}/fs/upload` | POST |上传文件（多部分，`?path=`）|
| `/opencomputers/hosts/{id}/fs/delete` | DELETE |删除文件/目录 (`?path=`) |
| `/opencomputers/hosts/{id}/fs/mkdir` | POST |创建目录(`{"path":"..."}`) |

**列出回复：**
```json
{
  "path": "/workspace/projects",
  "entries": [
    {
      "name": "my-app",
      "path": "/workspace/projects/my-app",
      "size": 0,
      "is_dir": true,
      "modified_at": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

## 终端

### 签发终端票

**`POST /api/v1/opencomputers/hosts/{id}/terminal/ticket`**

返回短期 WebSocket 身份验证票证。连接到 `ws_url`
立即使用票证作为查询参数。
```json
{
  "ticket": "tk_abc123",
  "ws_url": "wss://api.miosa.ai/opencomputers/ws/terminal?ticket=tk_abc123",
  "expires_at": "2026-01-01T00:00:30Z"
}
```

---

## 桌面（VNC）

### 发出桌面票证

**`POST /api/v1/opencomputers/hosts/{id}/desktop/ticket`**

与航站楼票形状相同。使用票证连接到 `ws_url`
启动 VNC 会话。

---

## 隧道

通过 MIOSA 管理的公共 URL 公开主机上的本地端口。

### 列出隧道

**`GET /api/v1/opencomputers/hosts/{id}/tunnels`**

### 创建隧道

**`POST /api/v1/opencomputers/hosts/{id}/tunnels`**

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `target_port` |整数 |是的 |暴露本地端口 |
| `auth_mode` |字符串|没有 | `public` \| `tenant_only` \| `password`（默认：`tenant_only`）|
| `slug` |字符串|没有 |公共 URL 的自定义 slug |
```json
{
  "id": "tun_abc",
  "host_id": "host_abc123",
  "slug": "my-app-dev",
  "target_port": 3000,
  "auth_mode": "tenant_only",
  "public_url": "https://api.miosa.ai/t/my-app-dev",
  "enabled": true,
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

### 获取/更新/删除隧道

**`GET /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}`**

**`PATCH /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}`**

|领域|类型 |描述 |
|--------|------|-------------|
| `target_port` |整数 |更改目标端口 |
| `auth_mode` |字符串|更改身份验证模式 |
| `enabled` |布尔 |启用或禁用隧道 |

**`DELETE /api/v1/opencomputers/hosts/{id}/tunnels/{tunnel_id}`** - `204 No Content`

---

## 智能体

调度AI智能体在主机上自主完成任务。

使用 `miosa api` 通过 API 进行调度：

```bash
miosa api POST /opencomputers/hosts/host_abc123/agent/dispatch \
  -d '{"task": "Audit the local browser workflow"}'
```

### 派遣智能体

**`POST /api/v1/opencomputers/hosts/{id}/agent/dispatch`**

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `task` |字符串|是的 |自然语言任务描述 |
| `model` / `model_id` |字符串|没有 |覆盖默认的运行时/模型 |
| `tools` |字符串[] |没有 |向智能体会话公开的工具名称 |
| `budget.max_steps` |整数 |没有 |最大智能体步数（默认：30）|
| `budget.max_tokens` |整数 |没有 |最大代币预算（默认：100000）|
| `budget.timeout_ms` |整数 |没有 |会话超时（以毫秒为单位）|
| `agent_runtime_profile_id` | UUID/字符串 |没有 |强制使用特定的运行时配置文件 |
| `skip_agent_runtime_profile` |布尔 |没有 |不应用租户/工作区默认配置文件 |
| `context` |对象|没有 |其他上下文键值对 |

```json
{
  "id": "sess_abc",
  "session_id": "sess_abc",
  "host_id": "host_abc123",
  "task": "Run the test suite and fix any failing tests",
  "status": "running",
  "sse_url": "/api/v1/opencomputers/hosts/host_abc123/agent/sessions/sess_abc/events",
  "optimal_session_id": "opt_sess_123",
  "agent_runtime_profile_id": "arp_123",
  "runtime_context": {
    "agent_runtime_profile": {
      "id": "arp_123",
      "runtime": "claude-code",
      "tools": ["browser", "filesystem"],
      "env_keys": ["ANTHROPIC_API_KEY"]
    }
  }
}
```

**会话状态：** `running` | `succeeded` | `failed` | `canceled`

### 列出/获取会话

**`GET /api/v1/opencomputers/hosts/{id}/agent/sessions`**

**`GET /api/v1/opencomputers/hosts/{id}/agent/sessions/{session_id}`**

### 流智能体事件 (SSE)

**`GET /api/v1/opencomputers/hosts/{id}/agent/sessions/{session_id}/events`**

### 取消会话

**`DELETE /api/v1/opencomputers/hosts/{id}/agent/sessions/{session_id}`** - `204 No Content`

---

## 推理集群

将多个主机分组，通过 OpenAI 兼容端点为 LLM 提供服务。

### 列出集群

**`GET /api/v1/opencomputers/clusters`**

### 创建集群

**`POST /api/v1/opencomputers/clusters`**

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `name` |字符串|是的 |集群名称 |
| `model` |字符串|是的 |服务模型（例如 `llama3:70b`）|
| `host_ids` |字符串[] |是的 |集群内主机ID |
```json
{
  "id": "cl_abc",
  "name": "my-cluster",
  "model": "llama3:70b",
  "slug": "my-cluster",
  "status": "active",
  "host_ids": ["host_abc123"],
  "inference_url": "https://api.miosa.ai/inference/my-cluster/v1",
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

`inference_url` 与 OpenAI 兼容：
```http

POST {inference_url}/chat/completions
```

### 获取/启动/停止/删除

**`GET /api/v1/opencomputers/clusters/{id}`**

**`POST /api/v1/opencomputers/clusters/{id}/start`**

**`POST /api/v1/opencomputers/clusters/{id}/stop`**

**`DELETE /api/v1/opencomputers/clusters/{id}`** - `204 No Content`

---

## 秘密

存储主机智能体在运行时可访问的加密键值秘密。

### 租户范围内的秘密

|端点|方法|描述 |
|----------|--------|-------------|
| `/opencomputers/secrets` | GET |列出租户秘密 |
| `/opencomputers/secrets` | POST |创造一个秘密|
| `/opencomputers/secrets/{id}` | PATCH |更新值或描述 |
| `/opencomputers/secrets/{id}` | DELETE |删除秘密 |

### 主机范围内的秘密

| 端点 | 方法 | 描述 |
|----------|--------|-------------|
| `/opencomputers/hosts/{id}/secrets` | GET |列出主机机密 |
| `/opencomputers/hosts/{id}/secrets` | POST |创建主机机密 |
| `/opencomputers/hosts/{id}/secrets/{secret_id}` | DELETE |删除主机机密 |

**创建请求：**
```json
{ "name": "GITHUB_TOKEN", "value": "ghp_xxx", "description": "CI token" }
```

**响应（永远不会返回值）：**
```json
{
  "id": "sec_abc",
  "name": "GITHUB_TOKEN",
  "description": "CI token",
  "host_id": null,
  "tenant_id": "t_abc",
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

---

## SDK 示例

**打字稿**

```typescript
import Miosa from 'miosa';

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY });

// Register a host  -  save host_key immediately
const host = await miosa.openComputers.hosts.create({ name: 'my-mac' });
console.log('Host key (save this!):', host.host_key);

// Run a command
const job = await miosa.openComputers.jobs.run(host.id, {
  command: 'npm test',
});
console.log('Exit code:', job.exit_code);
console.log('Output:', job.stdout);

// Expose a local dev server
const tunnel = await miosa.openComputers.tunnels.create(host.id, {
  target_port: 3000,
});
console.log('Public URL:', tunnel.public_url);

// Dispatch an AI agent
const session = await miosa.openComputers.agents.dispatch(host.id, {
  task: 'Run the test suite and fix any failing tests',
  max_turns: 30,
});
console.log('Session:', session.id, session.status);
```

**Python**

```python
import miosa

client = miosa.Miosa(api_key="msk_u_...")

# Register a host
host = client.open_computers.hosts.create(
    miosa.resources.open_computers.types.HostCreateParams(name="my-mac")
)
print("Host key:", host.host_key)

# Run a command
from miosa.resources.open_computers.types import JobRunParams
job = client.open_computers.jobs.run(host.id, JobRunParams(command="npm test"))
print(f"Exit code: {job.exit_code}, stdout: {job.stdout}")

# Create a tunnel
from miosa.resources.open_computers.types import TunnelCreateParams
tunnel = client.open_computers.tunnels.create(host.id, TunnelCreateParams(target_port=3000))
print("Public URL:", tunnel.public_url)
```

**去**

```go
client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))

// Register a host
host, err := client.OpenComputers.Hosts.Create(ctx, miosa.CreateHostInput{
    Name: "my-mac",
})
if err != nil { log.Fatal(err) }
fmt.Println("Host key:", *host.HostKey)

// Run a command
job, err := client.OpenComputers.Jobs.Run(ctx, host.ID, miosa.RunJobInput{
    Command: "npm test",
})
if err != nil { log.Fatal(err) }
fmt.Printf("Exit code: %d\nStdout: %s\n", *job.ExitCode, *job.Stdout)

// Create a tunnel
tunnel, err := client.OpenComputers.Tunnels.Create(ctx, host.ID, miosa.CreateTunnelInput{
    TargetPort: 3000,
})
fmt.Println("Public URL:", tunnel.PublicURL)
```

**长生不老药**

```elixir
client = Miosa.client("msk_u_...")

# Register a host
{:ok, host} = Miosa.OpenComputers.Hosts.create(client, %{name: "my-mac"})
IO.puts("Host key: #{host["host_key"]}")

# Run a command
{:ok, job} = Miosa.OpenComputers.Jobs.run(client, host["id"], %{command: "npm test"})
IO.puts("Exit code: #{job["exit_code"]}")

# Create a tunnel
{:ok, tunnel} = Miosa.OpenComputers.Tunnels.create(client, host["id"], %{target_port: 3000})
IO.puts("Public URL: #{tunnel["public_url"]}")
```

**Java**

```java
MiosaClient miosa = new MiosaClient(System.getenv("MIOSA_API_KEY"));

// Register a host
HostData host = miosa.openComputers().hosts()
    .create(new CreateHostParams("my-mac"));
System.out.println("Host key: " + host.hostKey);

// Run a command
JobData job = miosa.openComputers().jobs()
    .run(host.id, new RunJobParams("npm test"));
System.out.printf("Exit code: %d%nStdout: %s%n", job.exitCode, job.stdout);

// Create a tunnel
TunnelData tunnel = miosa.openComputers().tunnels()
    .create(host.id, new CreateTunnelParams(3000));
System.out.println("Public URL: " + tunnel.publicUrl);
```

---

## 常见错误

|状态 |代码|原因 |
|--------|------|--------|
| 404 | `NOT_FOUND` | 此租户中不存在主机或资源 |
| 403 | `FORBIDDEN` | 已认证但未授权 |
| 400 | `INVALID_ID` |路径参数不是有效的 ID |
| 409 |  -  | 主机离线；无法发送命令 |
| 502 |  -  | 主机智能体无法访问 |

---

## 另请参阅

- [计算机 API](/docs/api-reference/computers) - 托管 MIOSA 虚拟机与 BYOC 主机
- [Exec API](/docs/api-reference/exec) - 在托管计算机上执行
- [Regions](/docs/api-reference/regions) - 主机创建时接受的区域段
- [错误代码](/docs/api-reference/errors) - 完整的错误代码参考
