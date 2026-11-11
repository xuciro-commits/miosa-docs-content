---
slug: "fundamentals/events"
title: "活动（SSE）"
description: "使用一次性票证流式传输服务器发送的事件的进度 - 每个资源流、租户范围的流和事件目录。"
section: "API reference"
order: 12004
---

几个长时间运行的操作通过[服务器发送的事件](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)传输进度。资源的 SSE 流始终位于：
```http

GET /api/v1/{resource-type}/{id}/events
```

由于浏览器 `EventSource` API 不支持自定义标头，SSE 端点使用 **一次性票证** 进行身份验证，而不是 `Authorization` 标头：

**Python**

```python
# 1. Mint a ticket (server-side)
ticket = client.sandboxes.sse_ticket(sandbox_id="sbx_abc123")

# 2. Open the stream
import sseclient, urllib.request

url = f"https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/events?ticket={ticket.value}"
with urllib.request.urlopen(url) as resp:
    for event in sseclient.SSEClient(resp):
        print(event.data)
```

**打字稿**

```typescript
// 1. Mint a ticket (server-side)
const ticket = await miosa.sandboxes.sseTicket('sbx_abc123');

// 2. Open the stream (browser or Node)
const url = `https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/events?ticket=${ticket.value}`;
const source = new EventSource(url);
source.onmessage = (e) => console.log(JSON.parse(e.data));
```

**curl**

```bash
# 1. Mint a ticket
TICKET=$(curl -s -X POST \
  https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/sse-ticket \
  -H "Authorization: Bearer msk_u_..." | jq -r '.ticket')

# 2. Stream events
curl -N "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/events?ticket=$TICKET"
```

事件类型因资源而异。常见事件形状：
```jsonc
// Build log line
{ "type": "log", "stream": "stdout", "data": "Installing dependencies...", "ts": 1748390460 }

// Status change
{ "type": "status_changed", "previous": "building", "current": "ready", "ts": 1748390462 }

// Error
{ "type": "error", "code": "BUILD_FAILED", "message": "Exit code 1", "ts": 1748390465 }
```

## 沙盒生命周期事件

|活动 |当 |
|---|---|
| `sandbox.created` |沙盒资源已创建（尚未启动）|
| `sandbox.ready` |启动完成、可执行和文件端点可用 |
| `sandbox.error` |致命启动或运行时错误 |
| `sandbox.destroyed` |沙盒停止，资源释放 |
| `sandbox.share.created` |为沙盒铸造了共享代币 |
| `sandbox.share.revoked` |股票代币被撤销 |

## 部署生命周期事件

|活动 |当 |
|---|---|
| `deployment.created` |已创建部署对象 |
| `deployment.build_started` | Builder VM 已启动，构建正在进行中 |
| `deployment.build_succeeded` |发布产物准备就绪 |
| `deployment.build_failed` |构建退出非零 |
| `deployment.published` |活动版本更新，流量割断 |
| `deployment.rollback` |活动版本重置为之前的版本 |

## 租户范围的 SSE 流

要接收组织的所有生命周期事件而不轮询每个资源端点，请订阅租户流：

```bash
# Mint a ticket
TICKET=$(curl -s -X POST https://api.miosa.ai/api/v1/events/ticket \
  -H "Authorization: Bearer msk_u_..." | jq -r '.ticket')

# Stream all org events
curl -N "https://api.miosa.ai/api/v1/events?ticket=$TICKET"
```

租户流上的每个事件都包含 `resource_type` 和 `resource_id` 字段，以便您可以路由到正确的处理程序：
```jsonc
{
  "type": "sandbox.ready",
  "resource_type": "sandbox",
  "resource_id": "sbx_abc123",
  "external_user_id": "alice@example.net",
  "ts": 1748390462
}
```

有关完整的事件目录和过滤选项，请参阅[事件 (SSE) API 参考](/docs/api-reference/events)和[租户事件](/docs/api-reference/tenant-events)。
