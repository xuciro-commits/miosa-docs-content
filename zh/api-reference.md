---
slug: "api-reference"
title: "API 参考"
description: "MIOSA 平台的 REST API 参考 - 基本 URL、身份验证、请求/响应信封和端点指南。"
section: "API reference"
order: 12009
---

> **提示**
>
> 想要浏览并尝试端点而不编写代码？
> [交互式沙箱 API 参考](/docs/api) 呈现
> 与浏览器内请求运行程序一起列入白名单的公共 Sandbox V1 OpenAPI 合约。
> 使用下面的端点指南来获取更广泛的平台 API。

每个 MIOSA API 端点都是基于 HTTPS 的 REST，返回 JSON，并遵循一致的信封格式。 SDK 封装了这些端点 - 当您需要低级控制或想要从没有官方 SDK 的语言进行集成时，您可以直接使用它们。

## 基本网址
```text

https://api.miosa.ai/api/v1
```

## 身份验证

所有请求都需要持有者令牌：
```http
Authorization: Bearer msk_u_...
```

MIOSA API 密钥 (`msk_*`) 和短期 JWT 令牌均被接受。密钥的范围仅限于组织及其工作区/项目。有关范围详细信息，请参阅 [API 密钥](/docs/platform/api-keys)。

## 请求格式

- `Content-Type: application/json` 适用于所有请求主体。
- 文件上传使用`multipart/form-data`。
- 所有字段名称均为 **snake_case**。
- 突变请求（`POST`、`PUT`、`PATCH`、`DELETE`）应包含 `Idempotency-Key` 标头以启用安全重试：
```http
POST /api/v1/sandboxes
Authorization: Bearer msk_u_...
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "template": "miosa-sandbox",
  "workspace_slug": "acme-corp",
  "project_slug": "lead-magnet",
  "external_workspace_id": "acct_123",
  "external_user_id": "user-456",
  "external_project_id": "project_789"
}
```

使用 UUID v4 作为幂等性密钥。 24 小时内重复相同的键将返回原始响应，而无需重新执行突变。
该密钥还可以保证重试安全：放置结果不确定的幂等创建会用 `Retry-After` 回答可重试的 `503 SANDBOX_PLACEMENT_RECONCILING`，而不是终端 `409`，因此使用相同的密钥重试会解析到相同的沙箱。

## 响应信封

单一资源响应：
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "my-sandbox",
    "status": "running",
    "created_at": "2026-05-17T10:00:00Z"
  }
}
```

列表响应包含分页对象：
```json
{
  "data": [ { "id": "...", "name": "..." } ],
  "page": {
    "page": 1,
    "page_size": 20,
    "total": 150
  }
}
```

所有列表端点接受的分页参数：

|参数|类型 |默认 |最大|
|---|---|---|---|
| `page` |整数 | 1 |  - |
| `page_size` | 整数 | 20 | 100 |

## 错误信封

所有错误都遵循单一形状：
```json
{
  "error": {
    "code": "sandbox_not_found",
    "message": "No sandbox with id sbx_abc exists in this workspace.",
    "request_id": "req_01jv..."
  }
}
```

联系支持人员时请包含 `request_id`。

### HTTP 状态代码

|状态 |意义|
|---|---|
| 200 | 成功|
| 201 | 创建|
| 202 | 已接受 - 异步操作已开始|
| 400 |错误的请求 - 缺少或无效参数 |
| 401 | 未经授权 - 令牌无效或丢失|
| 403 | 禁止 - 有效令牌，范围不足|
| 404 | 没有找到|
| 409 | 冲突 - 无效状态转换（例如启动正在运行的计算机）|
| 413 | 负载太大 - 文件上传超过 10 MB|
| 422 | 无法处理的实体 - 验证错误|
| 429 | 限价|
| 500 | 内部服务器错误|
| 502 | 网关错误 - 命令服务无法访问|

## 速率限制

速率限制适用于每个 API 密钥、租户或组织以及端点系列。
当前的自助服务默认值为每分钟 600 个用户密钥请求和每分钟 30,000 个平台密钥请求。
特定端点系列和租户策略可以施加较低的限制。
将 `X-RateLimit-*` 和 `Retry-After` 响应标头视为权威。

## ID 和时间戳

当前API返回的规范资源ID是UUID。
一些较旧的 SDK 示例和兼容性表面可能仍会显示人类可读的别名。
传递 API 返回的确切 ID，而不是构造标识符或为其添加前缀。
所有时间戳均为 UTC 格式的 ISO 8601：`2026-05-17T10:00:00Z`。

## 所有权字段

为接受相同所有权选择器的资源创建端点：

|领域|使用 |
|---|---|
| `workspace_id` |现有 MIOSA 工作区 UUID |
| `workspace_slug` |现有或自动创建的工作区 slug |
| `workspace_name` |自动创建工作区时使用的显示名称 |
| `project_id` |现有 MIOSA 项目 UUID |
| `project_slug` |现有或自动创建的项目 slug |
| `project_name` |自动创建项目时使用的显示名称 |
| `external_workspace_id` |您的客户/帐户/工作空间 ID |
| `external_user_id` |您的最终用户 ID |
| `external_project_id` |您的项目/应用程序/文档 ID |

当资源归工作区/项目所有时，响应包括 `workspace_id` 和 `project_id`。请参阅[所有权和归属](/docs/platform/attribution)。

---

## 计算- **沙箱**：创建、列出、获取、停止和删除沙箱环境。核心计算原语。
  [打开→](/docs/api-reference/sandboxes)
- **智能体运行**：通过 OSA、Codex、Claude Code、Pi、Hermes 或自定义运行时将提示分派到沙箱或计算机中。
  [打开→](/docs/api-reference/agent-runs)
- **智能体运行时配置文件**：配置可重用智能体运行时、工具、连接器授权、策略和非秘密环境默认值。
  [打开→](/docs/api-reference/agent-runtime-profiles)
- **Exec**：在正在运行的计算机或沙箱中运行 bash 命令和 Python 代码。
  [打开→](/docs/api-reference/exec)
- **Streaming Exec**：服务器发送的用于长时间运行命令的事件流 - 逐行 stdout/stderr。
  [打开→](/docs/api-reference/streaming-exec)
- **文件**：读取、写入、统计、mkdir、重命名、复制和更改计算机内的文件。
  [打开→](/docs/api-reference/files)
- **文件系统**：目录列表、多文件操作和存档上传/下载。
  [打开→](/docs/api-reference/filesystem)
- **快照**：创建、恢复和删除计算机磁盘状态的时间点快照。
  [打开→](/docs/api-reference/snapshots)

## 部署

- **部署**：管理静态和服务器部署 - 创建、列出、获取和删除。
  [打开→](/docs/api-reference/deployments)
- **版本**：不可变的部署版本。每次发布都会创建一个新版本。
  [打开→](/docs/api-reference/versions)
- **发布**：将版本升级为指定版本（生产、暂存等）。
  [打开→](/docs/api-reference/releases)
- **自定义域**：注册、验证和管理部署上的自定义域。
  [打开→](/docs/api-reference/custom-domains)

## 计算机

- **计算机**：桌面虚拟机的完整生命周期 - 创建、启动、停止、重新启动、调整大小。
  [打开→](/docs/api-reference/computers)
- **桌面**：桌面控制操作：屏幕截图、单击、键入、按键、滚动、拖动、热键、窗口、辅助功能树。
  [打开→](/docs/api-reference/desktop)
- **OpenComputers**：将您自己的硬件注册为 MIOSA 计算机 (BYOC)。一个命令，任何机器。
  [打开→](/docs/api-reference/open-computers)

## 平台

- **工作空间**：组织内的客户/客户工作空间。
  [打开→](/docs/api-reference/workspaces)
- **项目**：工作区内的应用程序、网站、文档和工作流程。
  [打开→](/docs/api-reference/projects)
- **服务**：附加到计算机的托管后台服务（数据库、队列等）。
  [打开→](/docs/api-reference/services)
- **数据库**：配置托管数据库并操作生命周期、备份、恢复、分支、副本、凭证、日志和指标。
  [打开→](/docs/api-reference/databases)
- **网络策略**：计算机的入站和出站网络规则。
  [打开→](/docs/api-reference/network-policy)
- **事件**：计算机和沙箱发出的生命周期事件 - 用于 Webhook 和审核日志。
  [打开→](/docs/api-reference/events)
- **积分**：读取积分余额、列出使用交易并充值。
  [打开→](/docs/api-reference/credits)
- **区域**：列出可用区域及其当前容量。
  [打开→](/docs/api-reference/regions)

> **注意**
>
> SDK 覆盖范围因语言和版本而异。
> 使用适用于您的语言的当前 SDK 页面来确认助手是否存在。
> 您始终可以直接调用记录的 REST 端点。
