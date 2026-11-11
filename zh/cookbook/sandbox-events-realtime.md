---
slug: "cookbook/sandbox-events-realtime"
title: "实时沙盒事件"
description: "使用 HMAC 签名的 Webhook（服务器到服务器）或 SSE 事件流（进程内）对沙箱生命周期事件做出反应。涵盖两种模式、幂等性以及何时选择每种模式。"
section: "Platform builders"
order: 9017
---

`~10 min` `Python` `TypeScript`

> **目标**：使用 webhook（持久、服务器到服务器）或 SSE 流（短暂、进程内）对沙箱生命周期事件 - `sandbox.running`、`sandbox.destroyed`、`deployment.live` 等做出反应。
>
> **您将使用什么**：Webhooks、租户 SSE 事件流

## 两种模式 - 选择一种

| |网络钩子 |SSE直播 |
|---|---|---|
|交货| MIOSA 调用您的网址 |您的流程拉动 |
|耐用性|失败重试 |如果进程终止则丢失 |
|授权 | HMAC-SHA256 签名 |您的 `msk_*` 键 |
|延迟 |约 100 毫秒 | 〜50ms |
|最适合 |后台作业、计费触发器、Slack 警报 |仪表板 UI、开发时调试 |

两种模式都携带相同的事件负载。您可以同时使用两者 - webhooks 用于持久的服务器端反应，SSE 用于实时前端小部件。

## 模式 A - Webhooks

### 注册一个 webhook 端点

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

hook = client.webhooks.create(
    url="https://api.example.net/webhooks/miosa",
    events=[
        "sandbox.running",
        "sandbox.destroyed",
        "sandbox.error",
        "deployment.live",
        "deployment.failed",
    ],
    description="Acme sandbox lifecycle",
)

# IMPORTANT: save hook["secret"] immediately  -  it is shown only once
print(f"Webhook ID: {hook['id']}")
print(f"Secret (save this): {hook.get('secret')}")
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const hook = await client.webhooks.create({
  url: 'https://api.example.net/webhooks/miosa',
  events: [
    'sandbox.running',
    'sandbox.destroyed',
    'sandbox.error',
    'deployment.live',
    'deployment.failed',
  ],
  description: 'Acme sandbox lifecycle',
});

// IMPORTANT: save hook.secret immediately  -  shown only once
console.log('Webhook ID:', hook.id);
console.log('Secret (save this):', (hook as any).secret);
```

### 验证签名

每个交付都包含一个 `Miosa-Signature` 标头：`t=<unix-timestamp>,v1=<hex-digest>`。签名的有效负载是`<timestamp>.<raw-body>`。

**Python（FastAPI）**

```python
import hashlib, hmac, time, json
from fastapi import FastAPI, Request, HTTPException

app = FastAPI()
WEBHOOK_SECRET = os.environ["MIOSA_WEBHOOK_SECRET"]

def verify_signature(body: bytes, header: str, secret: str) -> bool:
    """Return True if the signature is valid and not a replay."""
    parts = dict(item.split("=", 1) for item in header.split(","))
    ts = parts.get("t", "")
    sig = parts.get("v1", "")

    # Replay guard: reject events older than 5 minutes
    if abs(time.time() - int(ts)) > 300:
        return False

    expected = hmac.new(
        secret.encode(),
        f"{ts}.{body.decode()}".encode(),
        hashlib.sha256,
    ).hexdigest()

    return hmac.compare_digest(expected, sig)

@app.post("/webhooks/miosa")
async def receive_webhook(request: Request):
    body = await request.body()
    header = request.headers.get("Miosa-Signature", "")

    if not verify_signature(body, header, WEBHOOK_SECRET):
        raise HTTPException(400, "Invalid signature")

    event = json.loads(body)
    event_type = event.get("type")

    # Idempotency: check event["id"] against your DB before processing
    event_id = event.get("id")

    if event_type == "sandbox.running":
        sandbox_id = event.get("data", {}).get("id")
        print(f"Sandbox {sandbox_id} is running  -  start your downstream job")

    elif event_type == "sandbox.destroyed":
        sandbox_id = event.get("data", {}).get("id")
        print(f"Sandbox {sandbox_id} destroyed  -  record billing event")

    elif event_type == "deployment.live":
        url = event.get("data", {}).get("url")
        print(f"Deployment live at {url}  -  notify user")

    # Always return 200 quickly; do heavy work asynchronously
    return {"ok": True}
```

**TypeScript（Express）**

```typescript
import express from 'express';
import { createHmac, timingSafeEqual } from 'crypto';

const app = express();
app.use(express.raw({ type: 'application/json' })); // raw body for signature check

const WEBHOOK_SECRET = process.env.MIOSA_WEBHOOK_SECRET!;

function verifySignature(body: Buffer, header: string, secret: string): boolean {
  const parts = Object.fromEntries(header.split(',').map(p => p.split('=')));
  const { t: ts, v1: sig } = parts;
  if (!ts || !sig) return false;

  // Replay guard: reject events older than 5 minutes
  if (Math.abs(Date.now() / 1000 - Number(ts)) > 300) return false;

  const expected = createHmac('sha256', secret)
    .update(`${ts}.${body.toString()}`)
    .digest('hex');

  return timingSafeEqual(Buffer.from(expected, 'hex'), Buffer.from(sig, 'hex'));
}

app.post('/webhooks/miosa', (req, res) => {
  const body = req.body as Buffer;
  const header = req.headers['miosa-signature'] as string ?? '';

  if (!verifySignature(body, header, WEBHOOK_SECRET)) {
    res.status(400).json({ error: 'Invalid signature' });
    return;
  }

  const event = JSON.parse(body.toString());
  const { type: eventType, id: eventId, data } = event;

  // Idempotency: check eventId against your DB before processing

  switch (eventType) {
    case 'sandbox.running':
      console.log(`Sandbox ${data?.id} is running  -  start downstream job`);
      break;
    case 'sandbox.destroyed':
      console.log(`Sandbox ${data?.id} destroyed  -  record billing event`);
      break;
    case 'deployment.live':
      console.log(`Deployment live at ${data?.url}  -  notify user`);
      break;
  }

  // Always return 200 quickly
  res.json({ ok: true });
});
```

### 幂等性

MIOSA 在失败时重试 Webhook 传递（5xx、超时）。您的处理程序必须是幂等的。最简单的方法是在处理之前将 `event.id` 保存在已看到事件表中：

**Python**

```python
import sqlite3

conn = sqlite3.connect("/var/lib/acme/events.db")
conn.execute(
    "CREATE TABLE IF NOT EXISTS seen_events (event_id TEXT PRIMARY KEY)"
)

def process_event(event: dict):
    event_id = event["id"]
    try:
        conn.execute("INSERT INTO seen_events VALUES (?)", (event_id,))
        conn.commit()
    except sqlite3.IntegrityError:
        # Already processed  -  skip
        return

    # Safe to process now
    handle_event(event)
```

**打字稿**

```typescript
const seenEvents = new Set<string>(); // use Redis or a DB table in production

function processEvent(event: Record<string, unknown>) {
  const eventId = String(event.id ?? '');
  if (seenEvents.has(eventId)) return; // already processed
  seenEvents.add(eventId);

  handleEvent(event);
}
```

## 模式 B - SSE 事件流

将租户 SSE 流用于进程内侦听器 - 仪表板后端、开发时间监控或您希望在正在运行的进程内做出反应而不是接收回调的任何场景。

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

# Stream all tenant-scoped events  -  runs until cancelled
for event in client.events.stream():
    event_type = event.get("type", "")

    if event_type == "sandbox.running":
        print(f"[running]   {event['data']['id']}")
    elif event_type == "sandbox.destroyed":
        print(f"[destroyed] {event['data']['id']}")
    elif event_type == "sandbox.error":
        print(f"[error]     {event['data']['id']}  -  {event['data'].get('error')}")
    elif event_type.startswith("deployment."):
        print(f"[deploy]    {event_type}  -  {event.get('data', {}).get('url', '')}")
```

**打字稿**

```typescript
import { Miosa, TenantEvents } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

// Stream all tenant-scoped events
// (TenantEvents is available as a standalone resource)
const eventStream = new TenantEvents((client as any).http);

for await (const event of eventStream.stream()) {
  const type = event.type ?? '';

  if (type === 'sandbox.running') {
    console.log(`[running]   ${(event as any).data?.id}`);
  } else if (type === 'sandbox.destroyed') {
    console.log(`[destroyed] ${(event as any).data?.id}`);
  } else if (type === 'sandbox.error') {
    console.log(`[error]     ${(event as any).data?.id}  -  ${(event as any).data?.error}`);
  } else if (type.startsWith('deployment.')) {
    console.log(`[deploy]    ${type}  -  ${(event as any).data?.url ?? ''}`);
  }
}
```

### 按事件类型过滤

该流接受 `types` 过滤器来减少噪音：

**Python**

```python
# Receive only deployment events
for event in client.events.stream(types=["deployment.live", "deployment.failed"]):
    print(event)
```

**打字稿**

```typescript
for await (const event of eventStream.stream(['deployment.live', 'deployment.failed'])) {
  console.log(event);
}
```

## 活动目录

|活动类型|触发|
|---|---|
| `sandbox.running` |沙盒启动完成 |
| `sandbox.paused` |沙盒暂停（空闲超时或手动暂停）|
| `sandbox.destroyed` |沙盒已删除 |
| `sandbox.error` |不可恢复的沙箱故障 |
| `sandbox.snapshot.ready` |快照上传完成 |
| `sandbox.share.created` |预览共享代币铸造 |
| `sandbox.share.revoked` |预览共享令牌被撤销 |
| `deployment.live` |部署升级上线 |
| `deployment.failed` |部署构建或升级失败 |

## 在模式之间进行选择

在以下情况下使用 **webhooks**：

- 你需要持久性（你的过程可能会重新启动）
- 您正在触发后台作业、计费事件或 Slack 通知
- 您希望 MIOSA 推动而不是流程拉动

在以下情况下使用 **SSE 流**：

- 您正在构建一个实时仪表板小部件
- 你正在编写一个短暂的脚本或开发工具
- 您想要最简单的集成，无需部署 Webhook 端点

## 相关

- [智能体构建应用程序](/docs/cookbook/agent-builds-app)：在智能体构建和部署时使用事件更新您的 UI。
- [每用户配额和计费](/docs/cookbook/per-user-quotas-billing)：对 sandbox.destroyed 事件触发计费操作。
- [Webhooks API 参考](/docs/api-reference/webhooks)：完整的事件目录、传递语义、重试策略和签名规范。
