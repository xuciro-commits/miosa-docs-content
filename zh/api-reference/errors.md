---
slug: "api-reference/errors"
title: "错误代码"
description: "MIOSA API 返回的每个错误代码、其含义以及如何处理它。"
section: "API reference"
order: 12011
---

大多数 MIOSA API 错误遵循单个信封形状 `{error: {code, message, request_id}}`。 HTTP状态表示大类； `code` 字段是机器可读的原因。一些系列有所不同：身份验证失败返回平面 `{error: {type, code}}` 形状，而快照错误可以返回平面消息字符串。

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "sandbox not found",
    "request_id": "req_01jv..."
  }
}
```

每当您联系支持人员时，请包括 `request_id`。它将所有内部服务的请求关联起来。

---

## 身份验证错误

在请求到达任何控制器之前由 `ApiKeyAuth` 插头返回。

|代码| HTTP |意义|该怎么办 |
|---|---|---|---|
| `invalid_token` | 401 | `Authorization` 标头丢失、格式错误或密钥/JWT 无效 | 验证标头为 `Authorization: Bearer msk_...` |
| `invalid_token` | 401 | 刷新非身份验证端点上使用的令牌 | 使用工作区 API 密钥 (`msk_*`)，而不是刷新令牌 |
| `insufficient_scope` | 403 | 密钥存在，但缺乏此操作所需的范围 | 调整仪表板中的关键范围 |
| `RATE_LIMIT_EXCEEDED` | 429 | 来自此密钥或 IP 的请求过多 | 后退并在 `Retry-After` 秒后重试 |

> **注意**
>
> 所有 API 端点均受到速率限制。
> 限制因密钥系列、租户策略和端点系列而异。
> 尊重 `X-RateLimit-*` 和 `Retry-After`，而不是硬编码客户端限制。

---

## 一般资源错误

这些代码出现在所有资源类型（计算机、沙箱、部署、工作区等）中。

|代码| HTTP |意义|该怎么办 |
|---|---|---|---|
| `NOT_FOUND` | 404 | 资源不存在或属于不同的租户 | 确认 ID 以及密钥的租户拥有该资源 |
| `FORBIDDEN` | 403 | 已通过身份验证但无权访问此资源 | 检查会员资格或所有权 |
| `INVALID_ID` | 400 |路径参数不是有效的 UUID |使用 UUID v4 ID，而不是别名或名称 |
| `MISSING_PARAM` | 400 |缺少必需的请求正文字段 |检查请求正文架构中的字段名称 |
| `VALIDATION_ERROR` | 422 | 一个或多个字段未通过架构验证 | 检查 `details` 中每个字段的消息 |
| `VALIDATION_FAILED` | 422 | 变更集验证失败（某些控制器中的 `VALIDATION_ERROR` 的别名） | 检查 `details` |
| `TENANT_RESOLUTION_FAILED` | 422 | 无法确定经过身份验证的租户 | API 密钥可能属于已删除或暂停的工作区 |
| `INTERNAL_ERROR` | 500 | 意外的服务器端错误 | 使用指数退避重试；如果持续存在则报告 |

---

## 沙盒错误

|代码 |HTTP |意义 |该怎么办 |
|---|---|---|---|
| `INSUFFICIENT_CREDITS` | 402 | 没有足够的积分来配置此沙箱 | 在仪表板中充值积分或减少资源请求 |
| `SANDBOX_LIMIT_REACHED` | 429 | 租户已达到其计划的并发沙盒上限（Developer 10、Business 250、Enterprise 500） | 销毁闲置沙箱、升级计划或联系支持人员提高限制 |
| `SANDBOX_NOT_RUNNING` | 409 | 操作要求沙箱处于`running`状态 | 等待状态为`running`；轮询 `GET /sandboxes/{id}` 或订阅事件 |
| `AGENT_UNAVAILABLE` | 502 | 计算机命令服务可访问但无响应 | 等待并重试；如果持续存在，则销毁并重新创建沙箱 |
| `INVALID_TEMPLATE` | 400 | `template_id` 不是可识别的模板 |使用 `GET /api/v1/sandbox-templates` 列出有效 ID |
| `MISSING_PATH` | 400 | `path` 字段是必需的但不存在 |在请求正文中包含 `path` |
| `INVALID_PATH` | 400 | `path` 不是有效的绝对文件系统路径 |使用绝对路径（以 `/` 开头）|
| `MISSING_FILE` | 400 |分段上传缺少 `file` 部分 |在多部分表单中包含文件部分 |
| `INVALID_BODY` | 400 |无法解析或解码请求正文 |验证 JSON 或 base64 编码 |
| `MISSING_MODE` | 400 | `chmod` 调用缺少 `mode` 字段 |包含八进制模式字符串，例如`"0644"` |
| `INVALID_PORT` | 422 | 暴露端口位于 `1`-`65535` | 外部使用有效范围内的端口 |
| `MISSING_PARAM` | 400 | `port` 在 `/expose` 上省略并且模板没有默认预览端口 |明确提供 `port` |
| `SANDBOX_NOT_RUNNING` | 409 | `/deploy` 已调用，但沙箱未运行 | 推广前确认沙箱状态 |
| `SANDBOX_PLACEMENT_UNCERTAIN` | 409 | 没有 `Idempotency-Key` 的创建无法确认其放置结果，因此沙箱可能存在也可能不存在 | 在创建另一个沙箱之前列出要协调的沙箱；没有密钥重试可能会导致重复 |
| `SANDBOX_PLACEMENT_RECONCILING` | 503 | 带有 `Idempotency-Key` 的创建尚无法确认其放置结果 | `Retry-After` 秒后使用相同的 `Idempotency-Key` 重试；重试解析到相同的沙箱 |
| `INVALID_SANDBOX_RUNTIME_PROFILE` | 422 | `runtime_profile` 不是 `standard`、`soma` 或 `auto` | 发送允许的值之一； `details.allowed` 命名它们 |
| `SOMA_RUNTIME_UNSUPPORTED_REQUEST` | 422 | 请求的形状高于运行时的最大值，或者运行时无法为其提供服务 | 请求较小的形状，或使用 `runtime_profile "standard"`； `details.max_shape` 命名上限 |

> **注意：幂等创建安全重试**
>
> 当沙箱创建无法确认其放置结果时，答案取决于您是否发送了 `Idempotency-Key` 标头。
>
> - 通过密钥，API 将返回可重试的 `503 SANDBOX_PLACEMENT_RECONCILING` 和 `Retry-After`。使用相同的密钥重试，它会解析为相同的沙箱。
> - 如果没有密钥，API 将返回终端 `409 SANDBOX_PLACEMENT_UNCERTAIN`。在创建另一个沙箱之前列出要协调的沙箱，因为重试可能会创建重复的沙箱。
>
> 每次创建时发送 `Idempotency-Key`，以便可以重试不确定的结果而不是终止。

---

## 沙盒模板构建错误

|代码| HTTP |意义|该怎么办 |
|---|---|---|---|
| `TEMPLATE_NOT_FOUND` | 404 | 模板ID不存在 | 验证模板ID |
| `BUILD_NOT_FOUND` | 404 | 构建记录不存在 | 验证构建 ID |
| `BUILD_ALREADY_TERMINAL` | 409 | 无法取消已经 `ready`、`failed` 或 `cancelled` | 的构建无需采取任何行动；构建处于最终状态 |
| `BUILD_NOT_RETRYABLE` | 409 | 构建未处于允许重试的状态（仅 `failed` 或 `cancelled`） | 只能重试 `failed` 或 `cancelled` 构建 |
| `UNAUTHORIZED` | 401 | SSE 票据缺失、无效或已过期 | 通过 `POST /auth/sse-ticket` 重新签发票据并重新连接 |

---

## 计算机错误

|代码 |HTTP |意义 |该怎么办 |
|---|---|---|---|
| `INSUFFICIENT_CREDITS` | 402 | 积分不足，无法启动这台计算机 | 充值积分 |
| `COMPUTER_NOT_RUNNING` | 409 | 桌面或执行操作需要一台运行中的计算机 | 先启动计算机 |
| `COMPUTER_NOT_FOUND` | 404 | 此租户中不存在计算机 | 验证计算机 ID |
| `AGENT_UNAVAILABLE` | 502 | 计算机命令服务无法访问 | 计算机可能正在启动；几秒钟后重试 |

---

## 部署错误

|代码| HTTP |意义|该怎么办 |
|---|---|---|---|
| `DEPLOYMENT_NOT_FOUND` | 404 | 部署 ID 不存在 | 验证部署 ID |
| `VALIDATION_ERROR` | 422 | 部署字段验证失败 | 检查 `details` 字段级消息 |
| `SANDBOX_NOT_RUNNING` | 409 | 发布需要运行沙箱 | 确认沙箱状态 |
| `RUNTIME_TARGET_UNAVAILABLE` | 409 | 沙箱还没有 IP - 无法路由到它 | 几秒钟后重试 |
| `RUNTIME_LOGS_UNAVAILABLE` | 502 | 无法从沙箱获取运行时日志 | 重试或检查沙箱运行状况 |
| `EMPTY_ARTIFACT` | 422 | 发布输出不包含文件 | 验证您的构建步骤生成输出文件 |
| `SERVICE_UNAVAILABLE` | 503 | 未配置所需的内部服务（例如加密） | 联系支持 |

---

## 快照错误

直接快照创建、列出、恢复和删除操作适用于 `/api/v1/computers/{id}/snapshots` 下的桌面计算机。
Canonical 沙箱 V1 在 `/api/v1/sandboxes/{id}/fork` 公开快照支持的分叉，而不是客户管理的快照 ID。

|代码| HTTP |意义|该怎么办 |
|---|---|---|---|
| `NOT_FOUND`（快照） | 404 | 快照不存在或属于另一台计算机 | 验证快照 ID 和计算机 ID |
| `FORBIDDEN` | 403 | 经过身份验证的租户不拥有此快照 | 检查所有权 |
| `already_deleted` | 409 | 快照已被删除 | 没有行动；已处于最终状态 |
| `snapshot is not in ready state` | 409 | 在快照达到 `ready` | 之前尝试恢复轮询快照状态直到 `ready` |
| `missing sse ticket` | 401 | SSE直播免票开启 | 先通过`POST /auth/sse-ticket`出票 |

---

## 自定义域错误

|代码| HTTP |意义|该怎么办 |
|---|---|---|---|
| `DOMAIN_NOT_FOUND` | 404 | 未找到自定义域记录 | 验证域名ID |
| `COMPUTER_NOT_FOUND` | 404 | 找不到此域所连接的计算机 | 验证计算机 ID |
| `FORBIDDEN` | 403 | 域属于不同的租户 | 验证所有权 |
| `VALIDATION_FAILED` | 422 | `fqdn` 缺失或无效 | 包含有效的完全限定域名 |

---

## 工作区错误

|代码| HTTP |意义|该怎么办 |
|---|---|---|---|
| `VALIDATION_FAILED` | 422 | 工作区创建或更新字段无效 | 检查 `details` |
| `TEMPLATE_NAME_TAKEN` | 409 | 此工作区中已存在具有该名称的模板 | 选择不同的名称 |

---

## 速率限制和配额错误

|代码| HTTP |意义|该怎么办 |
|---|---|---|---|
| `RATE_LIMIT_EXCEEDED` | 429 | 通用API速率限制命中 | 尊重 `Retry-After` 响应标头 |
| `SANDBOX_LIMIT_REACHED` | 429 | 并发沙箱达到上限 | 销毁闲置沙箱或升级计划 |
| `INSUFFICIENT_CREDITS` | 402 | 信用余额耗尽 | 从计费仪表板购买积分 |

> **注意**
>
> 所有 429 个响应均包含 `Retry-After` 标头（秒）。遵守它 - 重复违规可能会导致临时 IP 封禁。

---

## 另请参阅

- [API 参考概述](/docs/api-reference) - 基本 URL、身份验证和请求格式
- [Sandboxes](/docs/api-reference/sandboxes) - 沙箱生命周期和文件操作
- [计算机](/docs/api-reference/computers) - 计算机 CRUD 和桌面控制
- [部署](/docs/api-reference/deployments) - 部署和版本错误
