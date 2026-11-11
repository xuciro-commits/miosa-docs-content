---
slug: "fundamentals/requests"
title: "请求、限制和错误"
description: "幂等性密钥、速率限制和配额标头，以及 MIOSA 错误信封及其 HTTP 状态代码。"
section: "API reference"
order: 12003
---

## 幂等性

每个突变端点（`POST`、`PUT`、`PATCH`、`DELETE`）接受可选的 `Idempotency-Key` 标头。使用相同密钥重试请求是安全的 - MIOSA 返回原始响应而不重新执行操作。
```http
POST /api/v1/sandboxes
Authorization: Bearer msk_u_...
Idempotency-Key: create-sandbox-for-alice-2026-05-17
Content-Type: application/json

{ "template_id": "miosa-sandbox", "external_user_id": "alice" }
```

幂等键是：

- 范围仅限于您的工作区和端点路径。
- 自首次使用起**24 小时**内有效。
- 无论原始请求是成功还是失败都会存储。

如果第一个请求在您收到响应之前超时，请使用相同的密钥重试。如果操作完成，您将收到原始结果；如果操作仍在进行中，您将收到 `202 Accepted`。

> **提示：密钥生成**
>
> 使用与您自己的业务事件相关的确定性、防冲突密钥 - 例如，`"create-sandbox-{your_request_id}"`。 UUID 或请求正文的哈希值都可以。

## 速率限制

速率限制适用于每个 API 密钥、每个工作区和每个端点系列。每个响应都包含配额标头：
```http
X-RateLimit-Limit: 600
X-RateLimit-Remaining: 587
X-RateLimit-Reset: 1748390460
```

当前自助服务默认值：

|凭证或端点系列 |限制|窗口|
|---|---|---|
|用户 API 密钥 | 600 个请求 |每键每分钟|
|平台 API 密钥 | 30,000 个请求 |每键每分钟 |
| Auth (`/auth/login`, `/auth/refresh`) | 60 个请求 |每个 IP 每分钟 |

租户和端点系列策略可以施加较低的限制。
将响应标头视为权威。

当您超过限制时，API 将返回 `429 Too Many Requests`，其中包含 `Retry-After` 标头，其中包含配额重置之前的秒数。

大容量工作负载（AI 智能体管道每分钟创建许多沙箱）可以从 [平台仪表板 ](https://miosa.ai/dashboard/workspace/quota) 或联系支持人员请求增加配额。

## 错误

所有错误都使用一致的包络：
```json
{
  "error": {
    "code": "SANDBOX_NOT_FOUND",
    "message": "No sandbox with id sbx_abc123 exists in this workspace.",
    "request_id": "req_01hx9z..."
  }
}
```

|领域|描述 |
|---|---|
| `code` |机器可读常量。在 `switch` 语句中使用它，而不是 `message`。 |
| `message` |人类可读的解释。 API 版本之间可能会发生变化。 |
| `request_id` |联系支持或提交错误报告时请包含此内容。 |

HTTP 状态代码直接映射到错误类别：

|状态 |类别 |
|---|---|
| `400` |无效的请求正文或参数 |
| `401` |凭证缺失或无效 |
| `403` |有效凭证，权限不足 |
| `404` |找不到资源（或对您的工作区不可见）|
| `409` |冲突 - 资源已存在，具有不同主体的重复幂等键 |
| `422` |验证错误 - 请求解析正确但域规则失败 |
| `429` |限价|
| `5xx` | MIOSA 端故障 - 通过指数退避安全重试 |

完整的错误代码目录位于 [API 参考](/docs/api-reference)中。

- [API 参考](/docs/api-reference)：包含请求/响应架构和错误代码的完整端点参考。
