---
slug: "api-reference/sandbox-processes"
title: "沙箱进程API"
description: "启动、监视和流式传输沙箱内长时间运行的后台进程的日志。"
section: "API reference"
order: 12023
---

进程 API 允许您在沙箱内启动长时间运行的后台命令并检查其输出 - 对于启动开发服务器、观察程序或任何超出单个 exec 调用运行的进程非常有用。

基本路径：`/api/v1/sandboxes/{id}/processes`

> **注意**
>
> 进程由沙箱运行时管理，并在沙箱的生命周期内持续存在。它们不会跨快照或分叉保留。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `POST` | `/api/v1/sandboxes/{id}/processes` |启动进程 |
| `GET` | `/api/v1/sandboxes/{id}/processes` |列出进程 |
| `GET` | `/api/v1/sandboxes/{id}/processes/{pid}` |获取进程状态 |
| `DELETE` | `/api/v1/sandboxes/{id}/processes/{pid}` |停止进程 |
| `GET` | `/api/v1/sandboxes/{id}/processes/{pid}/logs` |获取最近的日志行 |
| `GET` | `/api/v1/sandboxes/{id}/processes/{pid}/stream` |SSE直播日志 |

---

## 启动一个进程

**`POST /api/v1/sandboxes/{id}/processes`**

### 授权

```http
Authorization: Bearer msk_...
```

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `cmd` |字符串|是的 |要运行的命令（传递给 `/bin/sh -c`）|
| `cwd` |字符串|没有 |工作目录（默认：`/workspace`）|
| `env` |对象|没有 |附加环境变量 |
| `name` |字符串|没有 |人类可读的标签 |

```json
{
  "cmd": "npm run dev -- --port 3000",
  "cwd": "/workspace",
  "name": "dev-server"
}
```

### 响应 - `201 Created`

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

## 列出进程

**`GET /api/v1/sandboxes/{id}/processes`**

### 响应 - `200 OK`

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

## 获取进程状态

**`GET /api/v1/sandboxes/{id}/processes/{pid}`**

### 响应 - `200 OK`

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

**进程状态值：**

|状态 |描述 |
|--------|-------------|
| `running` |进程还活着 |
| `exited` |进程正常退出 |
| `killed` |通过 DELETE | 停止进程
| `error` |进程启动失败 |

---

## 停止进程

**`DELETE /api/v1/sandboxes/{id}/processes/{pid}`**

在宽限期后发送 `SIGTERM`，然后发送 `SIGKILL`。

### 响应 - `204 No Content`

---

## 获取日志

**`GET /api/v1/sandboxes/{id}/processes/{pid}/logs`**

返回 stdout+stderr 的最后 N 行。

### 查询参数

|参数|类型 |默认 |描述 |
|------------|------|---------|-------------|
| `tail` |整数 | `200` |返回的行数 |

### 响应 - `200 OK`

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

## 实时日志流 (SSE)

**`GET /api/v1/sandboxes/{id}/processes/{pid}/stream`**

将 stdout+stderr 作为 SSE 进行流式传输。使用 `text/event-stream` - 需要承载令牌身份验证。

```text
id: log_001
event: log
data: {"line":"  ready in 743ms","ts":"2026-05-26T10:14:24Z"}

: heartbeat
```

---

## 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `INVALID_ID` |沙箱 ID 不是有效的 UUID |
| 404 | `NOT_FOUND`|沙箱或进程不存在|
| 409 | `SANDBOX_NOT_RUNNING`|沙箱必须处于 `running` 状态才能启动进程 |
| 500 | `PROCESS_START_FAILED`|运行时无法生成进程 |

---

## 示例

**curl**

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

**打字稿**

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
