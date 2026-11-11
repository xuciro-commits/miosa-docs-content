---
slug: "api-reference/tenant-events"
title: "租户事件 (SSE) API"
description: "订阅所有租户范围平台事件的实时服务器发送事件流。"
section: "API reference"
order: 12053
---

租户事件流为租户中的所有活动提供实时服务器发送事件 (SSE) - 沙箱生命周期更改、部署、Webhook 交付等。订阅一次即可接收所有资源的事件。

基本路径：`/api/v1/events/stream`

> **注意**
>
> 该端点使用 `text/event-stream` (SSE)。根据您的请求设置 `Accept: text/event-stream`，或使用本机 `EventSource` 客户端。

> **注意**
>
> 浏览器 `EventSource` 无法设置 `Authorization` 标头。使用服务器端客户端或传递从`POST /api/v1/auth/sse-ticket`获取的SSE票证以供浏览器使用。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/events/stream` |开放租户范围的 SSE 事件流 |

---

## 打开事件流

**`GET /api/v1/events/stream`**

升级到 SSE 流。连接保持打开状态，直到客户端断开连接或服务器将其关闭。每 15 秒发送一条 `: heartbeat` 注释，以通过智能体保持连接处于活动状态。

### 授权

```http
Authorization: Bearer msk_...
```

### 查询参数

|参数|类型 |必填 |描述 |
|------------|------|----------|-------------|
| `types` |字符串|没有 |用于过滤事件的逗号分隔的全局模式（例如 `sandbox.*,webhook.delivered`）|

### 响应 - `200 OK` (`text/event-stream`)

每个事件都是一个标准的SSE框架：

```text
id: evt_01hwxyz...
event: sandbox.ready
data: {"type":"sandbox.ready","id":"sbx_abc123","status":"running","tenant_id":"...","ts":"2026-05-26T10:14:23Z"}

id: evt_01hwyza...
event: deployment.succeeded
data: {"type":"deployment.succeeded","id":"dep_xyz456","url":"https://my-app.miosa.app","ts":"2026-05-26T10:15:01Z"}

: heartbeat
```

### 事件形状

所有事件共享一个共同的信封：

```json
{
  "type": "sandbox.ready",
  "id": "sbx_abc123",
  "tenant_id": "ten_...",
  "ts": "2026-05-26T10:14:23Z"
}
```

其他字段因事件类型而异（见下表）。

---

## 事件类型

### 沙盒活动

|类型 |描述 |额外字段 |
|------|-------------|-------------|
| `sandbox.created` |沙箱配置| `image`、`workspace_id` |
| `sandbox.ready` |沙盒达到 `running` 状态 | `status` |
| `sandbox.error` |沙盒启动失败| `error`、`reason` |
| `sandbox.destroyed` |沙盒已删除 |  - |
| `sandbox.paused` |沙箱 vCPU 暂停 |  - |
| `sandbox.resumed` |沙箱 vCPU 恢复 |  - |

### 电脑活动

|类型 |描述 |额外字段 |
|------|-------------|-------------|
| `computer.created` |计算机配置| `size`、`workspace_id` |
| `computer.running` |计算机达到 `running` 状态 |  - |
| `computer.stopped` |计算机停止运行 |  - |
| `computer.deleted` |电脑已删除 |  - |

### 部署事件

|类型 |描述 |额外字段 |
|------|-------------|-------------|
| `deployment.build_started` |构建作业排队 | `version_id` |
| `deployment.succeeded` |建设完成并推广 | `url`、`version_id` |
| `deployment.failed` |构建或升级失败 | `error` |
| `deployment.rollback` |版本回滚| `from_version`、`to_version` |

### Webhook 事件

|类型 |描述 |额外字段 |
|------|-------------|-------------|
| `webhook.delivered` |传出 webhook POST 成功 | `webhook_id`、`status_code` |
| `webhook.failed` |传出 webhook POST 失败 | `webhook_id`、`attempt`、`error` |

### 使用 `types` 进行过滤

传递以逗号分隔的全局模式列表。 `*` 通配符匹配任何后缀段：

```text
?types=sandbox.*           # all sandbox events
?types=sandbox.*,computer.running
?types=deployment.succeeded,deployment.failed
```

空的 `types` 参数（或无参数）传递所有事件。

---

## 重新连接

断开连接时，使用设置为您收到的最后一个 `id` 的 `Last-Event-ID` 标头重新连接。服务器将从该光标处恢复，重播最多 60 秒的积压事件。

```http
GET /api/v1/events/stream
Authorization: Bearer msk_...
Last-Event-ID: evt_01hwxyz...
```

---

## 示例

**curl**

```bash
# All events
curl -N -H "Authorization: Bearer msk_u_..." \
  -H "Accept: text/event-stream" \
  "https://api.miosa.ai/api/v1/events/stream"

# Sandbox events only
curl -N -H "Authorization: Bearer msk_u_..." \
  -H "Accept: text/event-stream" \
  "https://api.miosa.ai/api/v1/events/stream?types=sandbox.*"
```

**Python**

```python
import httpx

headers = {
    "Authorization": "Bearer msk_u_...",
    "Accept": "text/event-stream",
}

with httpx.stream("GET", "https://api.miosa.ai/api/v1/events/stream",
                  params={"types": "sandbox.*"}, headers=headers) as r:
    for line in r.iter_lines():
        if line.startswith("data:"):
            import json
            event = json.loads(line[5:].strip())
            print(event["type"], event["id"])
```

**打字稿**

```typescript
// Server-side (Node.js / Deno)  -  EventSource doesn't support custom headers in browsers
import EventSource from 'eventsource';

const es = new EventSource(
  'https://api.miosa.ai/api/v1/events/stream?types=sandbox.*',
  { headers: { Authorization: 'Bearer msk_u_...' } }
);

es.addEventListener('sandbox.ready', (e) => {
  const data = JSON.parse(e.data);
  console.log('Sandbox ready:', data.id);
});

es.addEventListener('sandbox.error', (e) => {
  const data = JSON.parse(e.data);
  console.error('Sandbox failed:', data.id, data.error);
});

es.onerror = () => es.close();
```
