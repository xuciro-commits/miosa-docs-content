---
slug: "integrations/webhooks"
title: "网络钩子"
description: "订阅沙箱和部署生命周期事件、验证 HMAC-SHA256 签名、处理重试和本地测试。"
section: "Platform builders"
order: 9012
---

每当沙箱或部署更改状态时，MIOSA 都会向您的端点发送 HTTP `POST` 请求。使用 Webhooks 更新数据库、通知用户、触发下游操作或计算费用。

---

## 事件分类

### 沙盒活动

|活动 |何时触发 |
|---|---|
| `sandbox.created` |沙箱行已插入。 VM 可能仍在启动。 |
| `sandbox.running` |沙箱已完全启动并准备就绪。 `external_user_id` 可用。 |
| `sandbox.paused` |由于空闲超时或显式 API 调用，沙箱已暂停。 |
| `sandbox.resumed` |暂停的沙箱已恢复。 |
| `sandbox.destroyed` |沙箱已被永久删除。 |
| `sandbox.error` |沙箱无法启动或遇到运行时错误。 |

### 部署事件

|活动 |何时触发 |
|---|---|
| `deployment.created` |创建了一个新的部署。 |
| `deployment.build_started` |新版本的构建开始了。 |
| `deployment.build_succeeded` |构建成功完成。 |
| `deployment.build_failed` |构建失败。 `error` 字段包含失败原因。 |
| `deployment.published` |版本已升级为活动版本。 |
| `deployment.domain_attached` |已验证并附加自定义域。 |

### 锻造事件

每个 Forge 事件都会在有效负载中为其存储库命名（`repository.id`、`repository.full_name`、`repository.owner`、`repository.slug`），因此一个组织订阅可以路由来自每个存储库的事件。订阅不能仅限于单个存储库。

|活动 |何时触发 |
|---|---|
| `forge.push` |已接受对存储库的推送。 |
| `forge.pull_request` |拉取请求被打开、关闭、重新打开、合并或审查。 |
| `forge.check` |对拉取请求或提交进行报告的检查。有效负载包括`commit_sha`。 |

---

## 负载结构

每个 webhook 传递都有相同的信封：

```json
{
  "id": "evt_01hzqmrkntq6g9gxnqhvpa8c7t",
  "type": "sandbox.running",
  "created_at": "2026-05-25T14:32:10Z",
  "data": {
    "id": "sbx_01hzq5pnmpgt6vdwp0r8d23c5n",
    "state": "running",
    "external_user_id": "user_42",
    "external_project_id": "proj_lumen_1",
    "metadata": {
      "customer_id": "user_42",
      "project_name": "brand-refresh",
      "source": "lumen-whitelabel"
    },
    "preview_url": "https://3000-sbx01hzq.sandbox.example.com"
  }
}
```

`id` 字段是唯一的事件 ID。用它来删除重复的交付。

---

## HMAC-SHA256签名验证

每个交付都包含一个 `Miosa-Signature` 标头：

```yaml
Miosa-Signature: t=1716645130,v1=a3f2c8...
```

- `t` - MIOSA 签署请求时的 Unix 时间戳（秒）
- `v1` - `"<t>.<raw_body>"` 的 HMAC-SHA256 十六进制摘要

签名的有效负载是`timestamp + "." + raw request body bytes`。在解析 JSON 之前进行验证。

> **注意：重放保护**
>
> 拒绝过去或未来时间戳 `t` 超过 300 秒（5 分钟）的事件。这可以防止使用捕获的有效签名进行重放攻击。

**Python**

```python
import hashlib
import hmac
import time

def verify_miosa_signature(body: bytes, header: str, secret: str) -> bool:
    """Return True if the signature is valid."""
    try:
        parts = dict(item.split("=", 1) for item in header.split(","))
        timestamp = parts["t"]
        received = parts["v1"]
    except (KeyError, ValueError):
        return False

    # Reject stale events.
    if abs(time.time() - int(timestamp)) > 300:
        return False

    signed = f"{timestamp}.{body.decode('utf-8')}"
    expected = hmac.new(
        secret.encode("utf-8"),
        signed.encode("utf-8"),
        hashlib.sha256,
    ).hexdigest()

    return hmac.compare_digest(expected, received)

# Flask example
from flask import request, jsonify

@app.post("/webhooks/miosa")
def miosa_webhook():
    raw = request.get_data()
    sig = request.headers.get("Miosa-Signature", "")

    if not verify_miosa_signature(raw, sig, WEBHOOK_SECRET):
        return jsonify({"error": "invalid signature"}), 401

    event = request.get_json()
    handle_event(event)
    return jsonify({"ok": True}), 200
```

**打字稿**

```typescript
import crypto from "node:crypto";
import type { Request, Response } from "express";

function verifyMiosaSignature(
  rawBody: Buffer,
  header: string,
  secret: string,
): boolean {
  const parts = Object.fromEntries(
    header.split(",").map((p) => p.split("=") as [string, string]),
  );
  const { t: timestamp, v1: received } = parts;
  if (!timestamp || !received) return false;

  // Reject stale events.
  if (Math.abs(Date.now() / 1000 - Number(timestamp)) > 300) return false;

  const signed = `${timestamp}.${rawBody.toString("utf8")}`;
  const expected = crypto
    .createHmac("sha256", secret)
    .update(signed)
    .digest("hex");

  return crypto.timingSafeEqual(
    Buffer.from(expected, "hex"),
    Buffer.from(received, "hex"),
  );
}

// Express example
app.post("/webhooks/miosa", express.raw({ type: "*/*" }), (req: Request, res: Response) => {
  const sig = req.headers["miosa-signature"] as string ?? "";
  if (!verifyMiosaSignature(req.body, sig, process.env.MIOSA_WEBHOOK_SECRET!)) {
    return res.status(401).json({ error: "invalid signature" });
  }
  const event = JSON.parse(req.body.toString());
  handleEvent(event);
  res.json({ ok: true });
});
```

**去**

```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "io"
    "math"
    "net/http"
    "strconv"
    "strings"
    "time"
)

func verifyMiosaSignature(body []byte, header, secret string) bool {
    parts := map[string]string{}
    for _, part := range strings.Split(header, ",") {
        kv := strings.SplitN(part, "=", 2)
        if len(kv) == 2 {
            parts[kv[0]] = kv[1]
        }
    }
    ts, ok1 := parts["t"]
    received, ok2 := parts["v1"]
    if !ok1 || !ok2 {
        return false
    }

    tsInt, err := strconv.ParseInt(ts, 10, 64)
    if err != nil || math.Abs(float64(time.Now().Unix()-tsInt)) > 300 {
        return false
    }

    signed := fmt.Sprintf("%s.%s", ts, string(body))
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write([]byte(signed))
    expected := hex.EncodeToString(mac.Sum(nil))

    return hmac.Equal([]byte(expected), []byte(received))
}

func webhookHandler(w http.ResponseWriter, r *http.Request) {
    body, _ := io.ReadAll(r.Body)
    sig := r.Header.Get("Miosa-Signature")

    if !verifyMiosaSignature(body, sig, webhookSecret) {
        http.Error(w, "invalid signature", http.StatusUnauthorized)
        return
    }

    // parse and handle body ...
    w.WriteHeader(http.StatusOK)
}
```

---

## 幂等性

MIOSA 可以多次传递同一事件（网络重试、至少一次语义）。通过存储事件 `id` 并丢弃重复项，使处理程序具有幂等性：

```python
processed_events = set()  # use a database or cache in production

def handle_event(event: dict) -> None:
    event_id = event["id"]
    if event_id in processed_events:
        return  # duplicate  -  skip
    processed_events.add(event_id)

    match event["type"]:
        case "sandbox.running":
            notify_user(event["data"]["external_user_id"])
        case "sandbox.destroyed":
            release_resources(event["data"]["id"])
        case "sandbox.error":
            alert_oncall(event["data"])
```

---

## 重试策略

|尝试|延迟 |
|---|---|
| 1（初始）|立即 |
| 2 | 1 分钟 |
| 3 | 5 分钟 |
| 4 | 30 分钟 |
| 5 | 1小时|
| 6 | 2小时|
| 7 | 4小时|
| 8 | 8小时|

MIOSA 在大约 15.5 小时内（24 小时窗口内）进行最多 8 次递送尝试，然后放弃该事件。如果您的端点无法访问，请检查 MIOSA 仪表板中的 Webhook 传送日志。

在 **10 秒**内返回 `2xx` 状态代码以确认收到。如果您的处理时间较长，请立即确认并异步处理该事件。

---

## 注册一个网络钩子

**Python**

```python
hook = client.webhooks.create(
    url="https://example.com/webhooks/miosa",
    events=[
        "sandbox.created",
        "sandbox.running",
        "sandbox.paused",
        "sandbox.destroyed",
        "sandbox.error",
    ],
    description="Lumen lifecycle events",
)

# The secret is returned only on creation. Store it immediately.
print(hook["secret"])  # MIOSA_WEBHOOK_SECRET=...
```

**打字稿**

```typescript
const hook = await client.webhooks.create({
  url: "https://example.com/webhooks/miosa",
  events: [
    "sandbox.created",
    "sandbox.running",
    "sandbox.paused",
    "sandbox.destroyed",
    "sandbox.error",
  ],
  description: "Lumen lifecycle events",
});

// The secret is returned only on creation. Store it immediately.
console.log(hook.secret); // MIOSA_WEBHOOK_SECRET=...
```

> **警告：立即复制机密**
>
> webhook 秘密在创建时返回一次。如果丢失，请删除该 Webhook 并创建一个新的。

---

## 本地测试

在开发过程中使用隧道公开本地服务器：

```bash
# Option 1  -  ngrok
ngrok http 8000
# Copy the https URL, set MIOSA_WEBHOOK_URL in .env

# Option 2  -  cloudflared
cloudflared tunnel --url http://localhost:8000
```

然后运行一次性设置脚本以将隧道 URL 注册为您的 Webhook 端点。当您重新启动隧道并获取新 URL 时，请更新 Webhook：

```python
# Update an existing webhook's URL
client.webhooks.update("whk_abc123", url="https://new-tunnel.ngrok.io/webhooks/miosa")
```

---

## 另请参阅

- [端到端白标演练](/docs/platform/white-label)
- [预览令牌](/docs/develop/previews)
- [沙盒 API 参考](/docs/api-reference/sandboxes)
