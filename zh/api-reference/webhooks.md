---
slug: "api-reference/webhooks"
title: "网络钩子 API"
description: "通过带有 HMAC 签名验证的传出 HTTP 回调来订阅平台生命周期事件。"
section: "API reference"
order: 12050
---

当发生平台事件（沙盒生命周期更改、部署完成、计费事件等）时，Webhook 会向您的端点发出传出 HTTP POST 请求。每份交付均使用 HMAC-SHA256 签名，以便您可以验证真实性。

基本路径：`/api/v1/webhooks`

> **注意**
>
> Webhook `secret` **仅返回一次** - 在对 `POST /webhooks` 的响应中。立即将其存放在安全的地方。稍后无法检索它（之后仅显示屏蔽预览）。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/webhooks` |列出所有 webhook |
| `POST` | `/api/v1/webhooks` |创建网络钩子 |
| `GET` | `/api/v1/webhooks/{id}` |获取网络钩子 |
| `PATCH` | `/api/v1/webhooks/{id}` |更新网络钩子 |
| `DELETE` | `/api/v1/webhooks/{id}` |删除网络钩子 |
| `POST` | `/api/v1/webhooks/{id}/test` |发送测试交付 |
| `GET` | `/api/v1/webhooks/{id}/deliveries` |列出递送尝试 |

---

## 列出 Webhooks

**`GET /api/v1/webhooks`**

### 授权

```http
Authorization: Bearer msk_...
```

### 响应 - `200 OK`

```json
{
  "data": [
    {
      "id": "wh_01hwxyz...",
      "name": "My Webhook",
      "url": "https://api.myapp.com/hooks/miosa",
      "events": ["sandbox.created", "sandbox.destroyed"],
      "secret_preview": "sk_live_••••••••abcd",
      "retry_count": 3,
      "created_at": "2026-05-01T00:00:00Z",
      "updated_at": "2026-05-01T00:00:00Z"
    }
  ]
}
```

---

## 创建一个网络钩子

**`POST /api/v1/webhooks`**

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `url` |字符串|是的 |将事件发布到的 HTTPS 端点 |
| `name` |字符串|没有 |人类可读的标签 |
| `events` |字符串[] |没有 |事件类型过滤器；空=所有事件|
| `headers` |对象|没有 |交付中包含的额外标头 |
| `retry_count` |整数 |没有 |在 webhook 被标记为 `failed` 之前连续失败的传递（0-10，默认 `3`） |
| `metadata` |对象|没有 |任意键值元数据 |

```json
{
  "url": "https://api.myapp.com/hooks/miosa",
  "name": "Sandbox lifecycle",
  "events": ["sandbox.created", "sandbox.ready", "sandbox.error", "sandbox.destroyed"],
  "retry_count": 3
}
```

### 响应 - `201 Created`

```json
{
  "data": {
    "id": "wh_01hwxyz...",
    "name": "Sandbox lifecycle",
    "url": "https://api.myapp.com/hooks/miosa",
    "events": ["sandbox.created", "sandbox.ready", "sandbox.error", "sandbox.destroyed"],
    "secret": "sk_live_abc123def456...",
    "secret_preview": "sk_live_••••••••3456",
    "retry_count": 3,
    "created_at": "2026-05-26T10:14:23Z",
    "updated_at": "2026-05-26T10:14:23Z"
  }
}
```

`secret` 字段仅出现在此响应中。安全保存。

---

## 更新 Webhook

**`PATCH /api/v1/webhooks/{id}`**

部分更新 - 仅包含您想要更改的字段。

### 请求正文

与创建相同的字段，都是可选的。

### 响应 - `200 OK`

不带 `secret` 字段的 Webhook 对象。

---

## 删除 Webhook

**`DELETE /api/v1/webhooks/{id}`**

### 响应 - `204 No Content`

---

## 发送测试交付

**`POST /api/v1/webhooks/{id}/test`**

将合成 `webhook.test` 事件发送到配置的 URL 以验证连接和签名验证。

该尝试像任何其他传递一样记录在传递日志中，标记为测试，因此端点收到的内容始终在 `GET /api/v1/webhooks/{id}/deliveries` 中可见。测试不计入 `retry_count`，并且永远不会将 Webhook 移动到 `failed`：针对暂时关闭的端点的测试不得暂停实时订阅。

### 响应 - `200 OK`

```json
{
  "delivered": true,
  "status_code": 200,
  "duration_ms": 143
}
```

---

## 列出交货

**`GET /api/v1/webhooks/{id}/deliveries`**

返回最近的传送尝试以及状态代码和时间。

### 响应 - `200 OK`

```json
{
  "data": [
    {
      "id": "del_01hwyza...",
      "event_type": "sandbox.ready",
      "status": "delivered",
      "status_code": 200,
      "attempt": 1,
      "duration_ms": 98,
      "delivered_at": "2026-05-26T10:14:24Z"
    },
    {
      "id": "del_01hwyzb...",
      "event_type": "sandbox.created",
      "status": "failed",
      "status_code": 500,
      "attempt": 3,
      "duration_ms": 5012,
      "delivered_at": "2026-05-26T10:14:20Z"
    }
  ]
}
```

---

## 签名验证

每个交付都包含一个 `X-Miosa-Signature` 标头。在信任有效负载之前，请在您的服务器上验证它：

```http
X-Miosa-Signature: sha256=<hex_digest>
```

计算预期签名：

```text
hmac = HMAC-SHA256(secret, raw_request_body_bytes)
expected = "sha256=" + hex(hmac)
assert constant_time_equals(expected, request.headers["X-Miosa-Signature"])
```

> **注意**
>
> 始终使用恒定时间函数来比较签名，以防止定时攻击。切勿使用 `==` 进行字符串比较。

### 验证示例（Python）

```python
import hmac, hashlib

def verify_signature(secret: str, body: bytes, header: str) -> bool:
    expected = "sha256=" + hmac.new(
        secret.encode(), body, hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, header)
```

### 验证示例（TypeScript）

```typescript
import { createHmac, timingSafeEqual } from 'crypto';

function verifySignature(secret: string, body: Buffer, header: string): boolean {
  const expected = 'sha256=' + createHmac('sha256', secret).update(body).digest('hex');
  return timingSafeEqual(Buffer.from(expected), Buffer.from(header));
}
```

---

## 重试策略

失败的交付将按照固定的退避时间表重试：

|尝试|延迟 |
|--------|--------|
| 1（初始）|立即 |
| 2 | 1 分钟 |
| 3 | 5 分钟 |
| 4 | 30 分钟 |
| 5 | 1小时|
| 6 | 2小时|
| 7 | 4小时|
| 8 | 8小时|

MIOSA 在大约 15.5 小时内（24 小时窗口内）进行最多 8 次递送尝试，然后放弃该事件。

如果您的终端返回非 2xx 状态码或 10 秒内未响应，则视为传送失败。 `retry_count`（0-10，默认 3）是 MIOSA 标记 webhook `failed` 后连续传递失败的次数；这不是每次传送重试上限。将其设置为 `0` 以标记 Webhook 在第一次失败传递时失败。

---

## 订阅的事件类型`GET /api/v1/webhooks/events` 返回此列表，按产品表面分组。 API 验证订阅时使用的是同一列表，因此无法订阅不在此的事件。

|活动 |描述 |
|--------|-------------|
| `sandbox.created` |沙盒已创建 |
| `sandbox.starting` |沙盒启动启动 |
| `sandbox.ready` |沙盒变得可用：`ready` 变为 true，无论这是在启动时还是由就绪探针决定的 |
| `sandbox.started` |沙箱运行时已启动且可访问 |
| `sandbox.paused` |沙箱左 `running` 为 `paused` |
| `sandbox.resumed` |暂停的沙箱又恢复了 |
| `sandbox.idle_paused` |沙箱已暂停，因为它在超时后处于空闲状态 |
| `sandbox.error` |沙盒无法启动或崩溃 |
| `sandbox.destroyed` |沙盒已删除 |
| `computer.created` |计算机创建 |
| `computer.started` |电脑已完成启动并可以使用 |
| `computer.stopped` |计算机被要求停止|
| `computer.deleted` |电脑被毁|
| `database.created` |数据库配置|
| `database.ready` |数据库接受连接 |
| `database.stopped` |数据库已停止 |
| `database.deleted` |数据库已删除 |
| `managed_database.provisioning` |托管数据库配置已启动 |
| `managed_database.running` |托管数据库运行 |
| `managed_database.failed` |托管数据库配置失败 |
| `managed_database.destroyed` |托管数据库被毁 |
| `managed_service.provisioning` |托管服务供应开始 |
| `managed_service.running` |托管服务运行 |
| `managed_service.failed` |托管服务配置失败 |
| `managed_service.destroyed` |托管服务被毁|
| `deployment.created` |部署已创建 |
| `deployment.building` |构建开始 |
| `deployment.build_started` |构建步骤开始执行 |
| `deployment.build_succeeded` |构建产生了一个产物|
| `deployment.build_failed` |构建失败 |
| `deployment.ready` |部署服务流量 |
| `deployment.error` |部署失败或崩溃 |
| `deployment.canceled` |部署已取消 |
| `deployment.rolled_back` |部署回滚到以前的版本 |
| `preview.created` |拉取请求预览已创建 |
| `preview.active` |预览服务流量 |
| `preview.destroyed` |预览被撕毁|
| `health_check.healthy` |端点健康检查已通过 |
| `health_check.unhealthy` |端点健康检查失败 |
| `cron.execution_started` |预定工作开始 |
| `cron.execution_completed` |预定工作完成 |
| `cron.execution_failed` |计划作业失败 |
| `billing.credits_low` |信用余额低于配置的阈值 |
| `billing.credits_exhausted` |信用余额为零 |
| `billing.sandbox.spend_threshold_reached` |沙盒支出已跨过门槛 |
| `run.created` |智能体运行已创建 |
| `run.queued` |智能体运行已排队 |
| `run.running` |智能体运行已开始 |
| `run.progress` |智能体运行报告进度 |
| `run.succeeded` |智能体运行成功 |
| `run.failed` |智能体运行失败 |
| `run.canceled` |智能体运行已取消 |
| `run.file_ready` |智能体运行生成的文件已准备就绪 |
| `forge.push` | Forge 存储库中的一个分支收到了提交 |
| `forge.pull_request` | Forge 拉取请求已打开、重新打开、关闭或合并 |
| `forge.check` |检查报告了 Forge 提交的结果 |

`forge.*` 有效负载携带 `repository_id`、`repository_slug` 和 `organization_slug`，因此一个订阅可以将每个事件路由到正确的存储库。

`webhook.test` 是 `POST /api/v1/webhooks/:id/test` 发送的合成事件。它不可订阅，并作为测试交付出现在交付日志中。

通过省略 `events` 字段来订阅所有事件。通过列出特定事件类型来订阅子集。

---

## 示例

**curl**

```bash
# Create
curl -X POST https://api.miosa.ai/api/v1/webhooks \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://api.myapp.com/hooks/miosa",
    "events": ["sandbox.created", "sandbox.destroyed"]
  }'

# Test
curl -X POST https://api.miosa.ai/api/v1/webhooks/wh_01hwxyz/test \
  -H "Authorization: Bearer msk_u_..."

# Deliveries
curl https://api.miosa.ai/api/v1/webhooks/wh_01hwxyz/deliveries \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

hook = client.webhooks.create(
    url="https://api.myapp.com/hooks/miosa",
    events=["sandbox.created", "sandbox.destroyed"],
    retry_count=3,
)

# Save hook.secret securely  -  only available now
print(hook.secret)

# Test
result = client.webhooks.test(hook.id)
print(result.delivered, result.status_code)
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

const hook = await client.webhooks.create({
  url: 'https://api.myapp.com/hooks/miosa',
  events: ['sandbox.created', 'sandbox.ready', 'sandbox.destroyed'],
  retryCount: 3,
});

// Store hook.secret in your secrets manager  -  only returned once
console.log(hook.secret);

// Test connectivity
const result = await client.webhooks.test(hook.id);
console.log(result.delivered, result.statusCode);
```
