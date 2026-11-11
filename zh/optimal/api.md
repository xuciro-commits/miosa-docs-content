---
slug: "optimal/api"
title: "最佳API"
description: "Optimal 的 HTTP 参考 - 创建 Optimal 令牌、创建聊天、发送消息、流式回复、绑定机器、上传附件、决定批准以及读取聊天费用。"
section: "Optimal"
order: 7010
---

最佳 API 与控制台使用的 API 相同。
控制台中的聊天所做的一切都从这些端点开始，您在此处创建的聊天会显示在控制台的**历史记录**中。

| | |
|---|---|
|基本网址 | `https://optimal.miosa.ai/api/v1` |
|授权 | `Authorization: Bearer <Optimal token>` |
|身体| JSON 与 `Content-Type: application/json`，附件上传除外 |
|错误 | `{"error": "<code>", "details": "<message>"}` |

## 身份验证

Optimal 不会直接获取您的 API 密钥。
您的 API 密钥会在 MIOSA API 上铸造一个短暂的 **Optimal 令牌**，并且每个 Optimal 调用都会使用该令牌。

API 密钥需要 **与 Optimal 聊天并开始运行** 权限 (`osa:write`)。
要使用令牌批准或拒绝 Optimal 的更改，密钥还需要 **批准或拒绝 Optimal 的更改** (`osa:approve`)。
绑定到一个工作区的密钥只能为该工作区创建一个令牌。

### 铸造一个工作区令牌

`POST https://api.miosa.ai/api/v1/optimal/workspace-token`

```bash
curl -s https://api.miosa.ai/api/v1/optimal/workspace-token \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workspace_id": "your_workspace_id"}'
```

```json
{
  "token": "eyJhbGciOi...",
  "expires_at": 1792771200,
  "can_decide_approvals": false,
  "scope": {
    "user_id": "...",
    "tenant_id": "...",
    "workspace_id": "your_workspace_id",
    "workspace_role": "admin"
  }
}
```

- 令牌持续10分钟； `expires_at` 是以秒为单位的 Unix 时间。
  当呼叫应答时创建一个新的 `401`。
- 聊天属于密钥的用户和令牌中的工作区。

|状态 |代码|当 |
|---|---|---|
| 401 | `UNAUTHORIZED` | API 密钥丢失或已撤销 |
| 403 | `FORBIDDEN` | 该键没有 `osa:write` |
| 403 | `WORKSPACE_FORBIDDEN` | 密钥的用户不是工作区的成员 |
| 403 | `API_KEY_WORKSPACE_BOUND` | 密钥绑定到不同的工作空间 |
| 403 | `API_KEY_NOT_ELIGIBLE` | 密钥绑定到项目或最终用户 |
| 422 | `WORKSPACE_REQUIRED` | 缺少 `workspace_id` |

### 铸造组织令牌

`POST https://api.miosa.ai/api/v1/optimal/org-token` 不带正文。
它为“所有工作区”聊天创建一个令牌，该令牌会读取用户所属的每个工作区，并询问在更改任何内容之前使用哪个工作区。
绑定到一个工作区的密钥无法铸造它。

## 聊天

聊天在 API 中称为会话。

### 创建聊天

`POST /sessions`

```bash
curl -s https://optimal.miosa.ai/api/v1/sessions \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"resume": false}'
```

```json
{
  "id": "your_chat_id",
  "status": "created",
  "machine": null,
  "workspace": { "id": "your_workspace_id", "rule": "This chat belongs to the workspace it was started in and never moves; its machines, memory and spend are that workspace's." },
  "pending_workspace": null,
  "turn": { "state": "idle", "turn_id": null, "started_at": null, "last_event_seq": 0 }
}
```

|领域|类型 |描述 |
|---|---|---|
| `resume` |布尔 |发送 `false` 始终开始新的聊天 |
| `client_key` |字符串|您自己的聊天密钥；使用相同的键再次创建将恢复它 (`status: "resumed"`) |
| `model` |字符串|选修的;唯一的值是 `optimal` |

### 列出聊天记录

`GET /sessions`

|查询 |描述 |
|---|---|
| `page`、`per_page` |分页； `per_page` 默认为 20，最多 100 |
| `view` | `mine`（默认），`team` 用于共享聊天，或 `members` 用于所有者和管理员 |

```json
{
  "sessions": [
    { "id": "your_chat_id", "title": "Landing page", "message_count": 6, "created_at": "2026-10-23T14:02:11Z", "visibility": "personal" }
  ],
  "count": 1,
  "page": 1,
  "per_page": 20
}
```

### 重命名或分享聊天

`PATCH /sessions/{id}` 与以下任意一项：

|领域|价值观 |
|---|---|
| `title` |文字|
| `mode` | `ask`、`build`、`investigate` |
| `visibility` | `personal`、`team` |

### 删除聊天记录

`DELETE /sessions/{id}` 永久删除聊天及其消息，并回复 `{"status": "deleted", "session_id": "..."}`。
聊天所使用的机器不会被触及。

## 消息

### 发送消息

`POST /sessions/{id}/message`

```bash
curl -s https://optimal.miosa.ai/api/v1/sessions/$CHAT/message \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "Why did my latest deployment fail?", "mode": "investigate", "client_message_id": "msg-001"}'
```

调用立即返回 `202`;回复到达 [stream](/docs/optimal/api#streaming)。

```json
{
  "status": "processing",
  "session_id": "your_chat_id",
  "turn_id": "msg-001",
  "client_message_id": "msg-001",
  "queued": false
}
```

|领域|类型 |描述 |
|---|---|---|
| `message` |字符串|必需的。要发送的文本 |
| `mode` |字符串| `ask`、`build` 或 `investigate` |
| `client_message_id` |字符串| 1 到 128 个字符的字母、数字、`_`、`.`、`:` 或 `-`。再次发送相同的 id 会用 `status: "duplicate"` 回答 `200`，而不是第二轮 |
| `machine` |对象| `{"resource_type": "sandbox" \| "computer", "resource_id": "..."}`：在该机器上运行此消息 |
| `confirm_switch` |布尔 | `true` 移动已在另一台计算机上运行的聊天 |
| `attachments` | 数组 | 来自 [附件](/docs/optimal/api#attachments) 的附件 ID |

Optimal 工作时发送的消息会在当前回合之后等待，并且 `queued` 为 `true`。
`DELETE /sessions/{id}/queued/{client_message_id}` 删除它直到它开始；之后呼叫应答 `409 not_queued`。

### 阅读文字记录

`GET /sessions/{id}/messages`

```json
{
  "messages": [
    { "role": "user", "content": "Why did my latest deployment fail?", "timestamp": "2026-10-23T14:02:11Z" },
    { "role": "assistant", "content": "The build failed at npm install...", "timestamp": "2026-10-23T14:02:40Z" }
  ],
  "count": 2,
  "queued": [],
  "owner": { "id": "...", "label": "Ada" }
}
```

### 停止回复

`POST /sessions/{id}/cancel` 回答 `{"status": "cancel_requested"}`。
当没有任何东西运行时，它会回答`404 not_running`。

## 流媒体

`GET /sessions/{id}/stream` 是服务器发送的事件流。

```bash
curl -N https://optimal.miosa.ai/api/v1/sessions/$CHAT/stream \
  -H "Authorization: Bearer $TOKEN"
```

- 浏览器`EventSource`无法设置标头，因此此端点也将令牌视为`?token=`。
  没有其他端点接受 URL 中的令牌。
- 每个事件都有一个编号 `id:`。
  重新连接 `Last-Event-ID` 标头或 `?last_event_id=`，以重播您错过的内容。
- 没有 id 的新读者从一开始就可以轮流飞行；完成的回合来自`GET /sessions/{id}/messages`。
- 如果您错过的事件消失了，流会发送 `replay_gap`；阅读文字记录，然后继续直播。

主要事件：

|活动 |数据|意义|
|---|---|---|
| `connected` | `session_id` |流已开放 |
| `turn_start` | `turn_id` |优化启动就留言|
| `streaming_token` | `text`、`message_id` |下一篇回复|
| `tool_call` | `name`、`phase`（`start` 或 `end`）|最好是使用工具 |
| `permission_required` |等待的步骤|某个步骤需要批准；请参阅[批准](/docs/optimal/api#approvals) |
| `agent_response` | `response` |回复完毕|
| `turn_billed` | `turn_credits`、`session_credits`、`balance` | 一旦账本结算，回合成本是多少 |
| `replay_gap` | `session_id`, `last_seq` |您要求重播的事件已消失 |

忽略您不使用的事件；可以添加新事件。

## 机器

聊天一次可以在工作区中的一台沙箱或一台计算机上进行。

`POST /sessions/{id}/machine`

```json
{ "resource_type": "sandbox", "resource_id": "your_sandbox_id" }
```

回答`{"session_id": "...", "machine": {"resource_type": "sandbox", "resource_id": "..."}}`。
`DELETE /sessions/{id}/machine` 解除绑定；机器继续运转。

|状态 |代码|当 |
|---|---|---|
| 400 | `invalid_machine` | `resource_type` 不是 `sandbox` 或 `computer`，或者缺少 `resource_id` |
| 404 | `machine_not_found` | 机器不在该工作空间中 |
| 409 | `machine_unavailable` | 机器未运行或无法访问 |
| 409 | `machine_switch_requires_confirmation` | 聊天正在另一台机器上进行；发送 `confirm_switch: true` |

## 附件

`POST /sessions/{id}/attachments?name={file_name}` 将原始文件字节作为正文。

```bash
curl -s "https://optimal.miosa.ai/api/v1/sessions/$CHAT/attachments?name=error.log" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/octet-stream" \
  --data-binary @error.log
```

```json
{ "id": "att_...", "name": "error.log", "mime": "text/plain", "bytes": 2048, "path": "/workspace/uploads/error.log", "vision": false }
```

将 `id` 传递到下一条消息的 `attachments` 中。
每个文件最多 10 MB，每个聊天最多 25 MB。
既不是图像也不是文档的文件需要与聊天绑定的机器，或者呼叫应答`409 no_machine`。

## 批准

`GET /approvals` 列出批准；使用 `session_id` 和 `status`（`pending`、`approved`、`rejected` 进行过滤。

```json
{ "approvals": [ { "id": "...", "status": "pending", "session_id": "your_chat_id", "title": "Delete sandbox api-old" } ], "count": 1 }
```

- `POST /approvals/{id}/approve` 批准； `POST /approvals/{id}/reject` 拒绝。
  两者都采用可选的 `reason`。
- `can_decide_approvals` 为 `false` 的标记获得 `403 approval_scope_required`。
- 决定两次答案`409 already_decided`。

请参阅[批准](/docs/optimal/approvals)。

## 型号

`GET /models` 返回一个模型 `optimal` 及其实时价格：

```json
{
  "models": [
    {
      "id": "optimal",
      "name": "Optimal",
      "provider": "MIOSA",
      "context_window": 1048576,
      "default": true,
      "credits_per_1m": { "input": 150, "output": 600 },
      "pricing": {
        "unit": "wallet credits (1 credit = 1 cent)",
        "input_credits_per_1m": 150,
        "output_credits_per_1m": 600
      }
    }
  ],
  "current": "optimal",
  "default": "optimal",
  "currency": "wallet credits (1 credit = 1 cent)"
}
```

任何其他 `model` 值都会回答 `400 MODEL_NOT_ALLOWED`。

## 一次聊天的费用

`GET https://api.miosa.ai/api/v1/credits/session/{chat_id}` 位于 MIOSA API 上并获取您的 API 密钥。

```json
{
  "session_id": "your_chat_id",
  "model_credits": 12,
  "machine_credits": 7,
  "total_credits": 19,
  "attributed": true,
  "settled": true,
  "as_of": "2026-10-23T14:10:00Z"
}
```

## 错误

|状态 |代码|意义|
|---|---|---|
| 400 | `missing or empty message` | `message` 为空 |
| 401 | `MISSING_TOKEN`，或过期的令牌 | 铸造新的 Optimal 令牌 |
| 402 | `OPTIMAL_CREDITS_REQUIRED` | 没有剩余积分；添加积分或升级 |
| 403 | `ownership_denied` | 该机器不在您的组织内 |
| 404 | `session_not_found` | 该聊天不存在，或者不是您的 |
| 415 | `unsupported_media_type` | 不带 `Content-Type: application/json` 的 JSON 正文 |
| 429 | `rate_limited` | 请求过多；等待 `Retry-After` |
| 429 | `session_ceiling` | 您的组织同时运行的聊天太多 |

另一个组织或工作区中的聊天回复 `404`，与不存在的聊天相同。

> **注意：仅聊天所需**
>
> 最佳令牌涉及聊天、历史记录、工件、调查、批准、成本和设置。
> 它无法直接运行工具或到达 MIOSA 自己的服务器。
> 请参阅[隔离和权限](/docs/optimal/isolation)。
