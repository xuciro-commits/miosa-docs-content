---
slug: "api-reference/services"
title: "服务API"
description: "用于管理 MIOSA 计算机内长期运行的 systemd 服务的 API 参考。"
section: "API reference"
order: 12033
---

服务是计算机内由 systemd 管理的持久进程。将它们用于 Web 服务器、后台工作程序、数据库或任何应在 exec 调用中存活并在失败时自动重新启动的守护程序。

基本路径：`/api/v1/computers/{id}/services`

> **注意**
>
> 服务直接映射到由计算机命令服务管理的 systemd 单元文件 (`/etc/systemd/system/miosa-{name}.service`)。计算机必须**运行**才能与服务交互。

---

## 快速入门
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Register and start a web server
const svc = await client.services.create(computerId, {
  name: 'webserver',
  command: 'python3 -m http.server 8080',
  cwd: '/workspace/app',
  restart: 'always',
});

console.log(svc.status); // "starting"

// Check status later
const updated = await client.services.get(computerId, svc.id);
console.log(updated.status); // "running"
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/services \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "webserver",
    "command": "python3 -m http.server 8080",
    "cwd": "/workspace/app",
    "restart": "always"
  }'
```

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `POST` | `/computers/{id}/services` |创建并启动服务 |
| `GET` | `/computers/{id}/services` |列出计算机上的所有服务 |
| `GET` | `/computers/{id}/services/{name}` |按名称获取服务 |
| `POST` | `/computers/{id}/services/{name}/start` |启动已停止的服务 |
| `POST` | `/computers/{id}/services/{name}/stop` |停止正在运行的服务 |
| `POST` | `/computers/{id}/services/{name}/restart` |重启服务 |
| `DELETE` | `/computers/{id}/services/{name}` |删除服务 |

---

## 创建服务

**`POST /api/v1/computers/{id}/services`**

写入 systemd 单元文件并启动服务。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `name` |字符串|是的 |服务名称。模式：`[a-z][a-z0-9-]{0,62}` |
| `command` |字符串|是的 |完整的运行命令 |
| `env` |对象|没有 |环境变量为 `{ "KEY": "value" }` |
| `cwd` |字符串|没有 |工作目录（默认：`/workspace`）|
| `restart` |字符串|没有 | `"always"`、`"on-failure"`（默认）或 `"no"` |

### 响应 - `201 Created`
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "computer_id": "...",
    "tenant_id": "...",
    "name": "webserver",
    "command": "python3 -m http.server 8080",
    "env": {},
    "cwd": "/workspace/app",
    "restart": "always",
    "status": "starting",
    "pid": null,
    "exit_code": null,
    "last_started_at": null,
    "last_exited_at": null,
    "inserted_at": "2026-04-11T00:00:00Z",
    "updated_at": "2026-04-11T00:00:00Z"
  }
}
```

### 状态值

|状态 |描述 |
|--------|-------------|
| `stopped` |已注册但尚未启动 |
| `starting` |开始分派到 systemd |
| `running` | systemd 报告 `ActiveState=active` |
| `failed` | systemd 报告 `ActiveState=failed` |
| `removed` |单位文件已删除；保留审计记录|

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 409 | `a service with this name already exists`|每台计算机的名称必须是唯一的 |
| 422 | 验证错误|名称格式无效或重新启动策略 |
| 502 | `AGENT_UNAVAILABLE`|虚拟机内智能体无法访问 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/services \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "api-server",
    "command": "/workspace/app/server",
    "env": {"PORT": "3000", "NODE_ENV": "production"},
    "cwd": "/workspace/app",
    "restart": "on-failure"
  }'
```

---

## 列出服务

**`GET /api/v1/computers/{id}/services`**

### 响应 - `200 OK`
```json
{
  "data": [
    {
      "id": "...",
      "name": "webserver",
      "status": "running",
      "pid": 12345,
      "restart": "always",
      "inserted_at": "2026-04-11T00:00:00Z"
    }
  ]
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/services \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取服务

**`GET /api/v1/computers/{id}/services/{name}`**

### 路径参数

|参数|类型 |描述 |
|------------|------|-------------|
| `id` | UUID |计算机 ID |
| `name` |字符串|服务名称|

### 响应 - `200 OK`

完整的服务对象（与创建响应相同）。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 404 | `service not found`|此计算机上不存在 |

---

## 启动、停止、重新启动

**`POST /api/v1/computers/{id}/services/{name}/start`**
**`POST /api/v1/computers/{id}/services/{name}/stop`**
**`POST /api/v1/computers/{id}/services/{name}/restart`**

所有返回一个带有更新的服务对象的 `200 OK`。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 409 | `service is already running`|无法启动正在运行的服务 |
| 409 | `service is not running`|无法停止未运行的服务 |
| 502 | `AGENT_UNAVAILABLE`|虚拟机内智能体无法访问 |
```bash
# Stop a service
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/services/webserver/stop \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Restart it
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/services/webserver/restart \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 删除服务

**`DELETE /api/v1/computers/{id}/services/{name}`**

停止服务并删除 systemd 单元文件。数据库记录转换为 `"removed"` 状态。

### 响应 - `200 OK`
```json
{
  "data": { "deleted": true }
}
```
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/services/webserver \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 常见食谱

### 将 Node.js 应用程序部署为服务
```typescript
// Upload app files first
await client.files.write(computerId, {
  path: '/workspace/app/server.js',
  content: serverJsContent,
});

// Register and start
const svc = await client.services.create(computerId, {
  name: 'node-app',
  command: 'node /workspace/app/server.js',
  env: { PORT: '3000' },
  restart: 'on-failure',
});
```

### 等待服务运行
```python
import time

svc = client.services.create(computer_id, name="worker", command="/usr/bin/worker")
while svc.status == "starting":
    time.sleep(1)
    svc = client.services.get(computer_id, svc.name)

if svc.status != "running":
    raise RuntimeError(f"Service failed to start: {svc.status}")
```

### 检查队列中失败的服务
```typescript
const computers = await client.computers.list();
for (const computer of computers.data) {
  if (computer.status !== 'running') continue;
  const services = await client.services.list(computer.id);
  const failed = services.data.filter(s => s.status === 'failed');
  if (failed.length > 0) {
    console.log(`${computer.name}: ${failed.length} failed services`);
  }
}
```
