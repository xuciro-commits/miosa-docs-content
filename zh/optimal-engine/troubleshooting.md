---
slug: "optimal-engine/troubleshooting"
title: "Optimal Engine 故障排查"
description: "常见 Optimal Engine 问题的修复方法——端口无监听、健康检查、密钥被拒、搜索为空、混合搜索不可用、存储正忙、迁移失败、Docker 健康检查，以及 MIOSA 上托管引擎的报错。"
section: "Optimal Engine"
order: 8011
---

每条先说明你会看到的现象，再给出修复方法。

## 启动

### “Optimal Engine needs Erlang/OTP 27 or later”（需要 Erlang/OTP 27 或更高版本）

该引擎使用 OTP 27 自带的 JSON 模块。
请安装 OTP 27 或更新版本；避免使用 28.0 至 28.2。

### 引擎启动了，但 4200 端口没有监听

除非设置 `OPTIMAL_API_ENABLED=true`，否则 HTTP API 处于关闭状态。
设置该变量后重新启动。
单独运行 `mix optimal.api` 只会打印一行监听信息，并不会开启 API，因此也要为它设置该变量。

### “OPTIMAL_AUTH_REQUIRED must be true or false”

某个设置的值不被引擎接受，启动因此中止。
错误信息会指出对应的变量；请修正它的值。
`OPTIMAL_EMBED_URL`、`OPTIMAL_SIZE_PROFILE`、`OPTIMAL_ENGINE_SCOPE` 以及[安装和配置](/docs/optimal-engine/install#settings)中的其他设置也是如此。

### “migration ... failed and was rolled back”（迁移失败并已回滚）

升级过程中的迁移失败了。
在修复之前，引擎不会打开存储。
请恢复升级前所做的备份，然后启动上一个版本；参见[运维](/docs/optimal-engine/operations#upgrades)。

### Docker 容器一直处于“unhealthy”状态

健康检查会使用 `OPTIMAL_ENGINE_API_KEY` 调用 `/api/health`，而镜像中要求提供密钥。
在容器上把 `OPTIMAL_ENGINE_API_KEY` 设置为有效密钥。

## 健康检查

`GET /api/health` 会报告 `status` 以及支撑该状态的各项检查。

| 警告或错误 | 修复方法 |
|---|---|
| `store_unreachable` | 数据库文件打不开。请检查 `OPTIMAL_ENGINE_DB`、磁盘和文件权限。 |
| `migrations_pending` | 重启引擎以应用迁移，或运行 `mix optimal.migrate`。 |
| `connector_key_missing` | 你在没有设置 `CONNECTOR_KEY` 的情况下配置了连接器或复制对端。请将其设置为 64 个十六进制字符。 |
| `auth_open_on_network` | 引擎绑定到了回环地址之外，且密钥校验关闭。请设置 `OPTIMAL_AUTH_REQUIRED=true`。 |
| `embedder_unreachable`、`embedder_unauthorized`、`embedder_dimension_mismatch` | 使用 `?full=true` 时显示。请检查 `OLLAMA_HOST` 或 `OPTIMAL_EMBED_URL`、其密钥以及 `OPTIMAL_EMBED_DIMENSIONS`。 |

## 请求

### 所有调用（包括 `/api/health`）都返回 401 `missing_api_key`

密钥是必需的。
发送 `Authorization: Bearer <key>`。
使用 `mix optimal.auth mint` 生成第一个密钥，或设置 `OPTIMAL_BOOTSTRAP_KEY`。

### 403 `workspace_scope_denied`

该密钥不覆盖该工作区。
使用 `GET /api/auth/whoami` 查看其工作区，并生成一个包含该工作区的密钥。

### 400 `workspace_required`

该密钥仅限指定的工作区，而请求中未指定任何工作区。
传入 `workspace`，或在 CLI 中使用 `-w`。

### 搜索刚存储的内容却没有任何结果

- 在每次调用中都使用相同的完整工作区 ID，例如 `default:notes`。
  引擎不会查找 `notes` 这样的裸 slug；它会将其视为另一个工作区 ID。
- `assemble` 生成的上下文包只包含已审核的事实；新来源需要在其声明被提升后才会出现在那里。
  使用 `search` 可以立即看到来源。

### 503 `hybrid_unavailable`

`mode=hybrid` 需要嵌入模型，但没有任何模型响应。
使用 `mode=auto`（会回退到全文搜索），或配置嵌入模型；参见[安装和配置](/docs/optimal-engine/install#embeddings)。

### 503 `busy`，带有 `Retry-After: 1`

存储正忙，通常发生在刚启动后或长时间写入期间。
请在延迟时间过后重试。

### 429 `rate_limited`

该密钥已用完本分钟的配额。
等待 `retry_after_ms`，或提高规格配置或该密钥的 `rate_limit_per_minute`；参见[运维](/docs/optimal-engine/operations#sizing)。

### 426 `client_incompatible`

`optimal` CLI 或 MCP 服务器与引擎的主版本号不同。
请将客户端更新到与引擎相同的发布版本。

## 在 MIOSA 上

### “Memory could not be started”（Memory 未能启动）

你的组织尚未启用托管引擎，或你的角色无权创建数据库。
参见[可用性](/docs/optimal-engine/miosa#availability)。

### 422 `ENGINE_NOT_AVAILABLE`

你的组织尚未启用托管引擎。
请联系支持人员开启。

### 409 `ENGINE_NOT_RUNNING`

引擎已停止或仍在启动中。
在**数据库**页面启动它，或创建后等待约一分钟。

### engine-api 智能体返回 404 `NOT_FOUND`

该路由不是客户端密钥可调用的路由，或者该工作区没有引擎。
允许的路由见[MIOSA 上的 Optimal Engine](/docs/optimal-engine/miosa#call-the-engine-api)。
密钥、备份、同步和维护路由则通过 MIOSA 管理。

### 客户端密钥停止工作

客户端密钥会过期，默认有效期为 24 小时。
再次运行 `miosa engine connect <engine>`，并重启 MCP 服务器（它在启动时只读取一次密钥）。
