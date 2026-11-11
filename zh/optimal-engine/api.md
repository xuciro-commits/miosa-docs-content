---
slug: "optimal-engine/api"
title: "Optimal Engine API 参考"
description: "Optimal Engine HTTP API - 基本 URL、键、范围、分页、速率限制、错误以及工作区、摄取、搜索、询问、内存、声明和操作的主要路由，以及curl 示例。"
section: "Optimal Engine"
order: 8008
---

| | |
|---|---|
|基本网址 | `http://127.0.0.1:4200/api` 在自承载引擎上。 `/v1/...` 是 `/api/...` 的别名。 |
|关于 MIOSA | `https://api.miosa.ai/api/v1/engines/<engine id>/engine-api/api/...` 带有引擎客户端密钥；请参阅[MIOSA 上的Optimal Engine](/docs/optimal-engine/miosa#call-the-engine-api) |
| API版本| `v1` |
|授权 | `Authorization: Bearer <key>` 或 `X-API-Key: <key>` |
|身体| JSON 与 `Content-Type: application/json` |
|错误 | `{"error": "<code>"}`，有时带有 `detail` |

引擎有大约 300 条路线。
此页面涵盖了大多数应用程序和智能体使用的页面。
完整的列表，以及每条路由所需的授权，从路由器生成到 [`docs/reference/api.md`](https://github.com/Miosa-osa/OptimalEngine/blob/main/docs/reference/api.md) 中，只要两者不一致，测试就会失败。

```bash
E=http://127.0.0.1:4200
K="Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
```

## 键和错误

当`OPTIMAL_AUTH_REQUIRED=true`时，每条路线都需要一个密钥，包括`/api/health`。

|状态 | `error` |当 |
|---|---|---|
| 400 | `invalid_json`、`invalid_parameter`、`workspace_required` |请求格式错误，或者仅限于命名工作区的密钥未命名 |
| 401 | `missing_api_key`、`invalid_api_key`、`api_key_revoked`、`api_key_expired` | 钥匙丢失或无法使用 |
| 403 | `workspace_scope_denied` | 关键不包括该工作区 |
| 403 | `permission_scope_denied` | 密钥不具有路由所需的范围 |
| 404 | `not found`, `workspace not found` | 没有这样的路线，或其他租户的工作空间 |
| 413 | `request_too_large` | 正文超过 8 MB |
| 426 | `client_incompatible` | 另一个主要版本的 `optimal` CLI 或 MCP 服务器 |
| 429 | `rate_limited` | 超过密钥的速率限制；正文有 `retry_after_ms` |
| 503 | `busy` | 商店很忙，例如刚开机后；请在 `Retry-After` 之后重试 |

### 速率限制

每个密钥都有自己的每分钟预算，由[大小配置](/docs/optimal-engine/operations#sizing)设置：默认 `business` 配置文件上每分钟 1,200 个请求。
没有有效密钥的请求共享其 IP 地址的预算。
每个响应都包含 `X-RateLimit-Limit`、`X-RateLimit-Remaining` 和 `X-RateLimit-Reset`。
`/api/health`、`/api/status` 和 `/api/metrics/prometheus` 不计算在内。

### 分页

列出采用 `offset` 和 `limit` 的路由（1 到 200，默认 50）并回答：

```json
"pagination": {"offset": 0, "limit": 50, "total": 132, "has_more": true}
```

## 引擎

|路线 |返回|
|---|---|
| `GET /api/health` | `status`（`up`、`degraded`、`down`）、`checks`、`warnings`，以及释放。 `?full=true` 为一个工作区运行十个商店诊断。 |
| `GET /api/status` |同样的，加上`api_version` |
| `GET /api/version` | `version`、`git_sha`、`build_timestamp`、`release_channel`、`api_version`、`expected_migration`、`capabilities` |
| `GET /api/metrics` | JSON 计数器、延迟和大小计量器 |
| `GET /api/metrics/prometheus` |普罗米修斯文本格式|
| `GET /api/auth/whoami` |键自己的作用域和工作区 |

## 工作空间和节点

### 创建工作区

`POST /api/workspaces`

```bash
curl -s -X POST $E/api/workspaces -H "$K" -H "Content-Type: application/json" \
  -d '{"slug": "sales", "name": "Sales"}'
```

|领域|必填 |意义|
|---|---|---|
| `slug` |是的 |小写名字，在租户中唯一 |
| `name` |是的 |显示名称 |
| `description` |没有| |
| `external_id` |没有|您自己的此工作空间的 ID，在租户中是唯一的 |

`201` 返回工作空间：`id` (`default:sales`)、`tenant_id`、`slug`、`name`、`status`、`created_at`。
错误：`400` 缺少或无效的 slug 或名称、`409 slug_taken`、`409 external_id_taken`。

|路线 |它有什么作用 |
|---|---|
| `GET /api/workspaces` |列出工作区，分页 |
| `GET /api/workspaces/:id`、`PATCH /api/workspaces/:id` |读取或重命名 |
| `POST /api/workspaces/:id/archive` |将其存档；其数据被保留|
| `GET /api/workspaces/:id/config`、`PATCH ...` |工作区设置 |

### 创建节点

`POST /api/nodes` 与 `{"name", "kind", "slug", "workspace", "description"}` 回答 `{"ok": true, "id", "slug", "kind"}`。
`kind` 是[概念](/docs/optimal-engine/concepts#topology)中的节点类型之一。

## 摄取

`POST /api/ingest` 将文本存储为源。
请参阅[来源和摄取](/docs/optimal-engine/ingestion)了解字段、文件、聊天会话、概念和连接器。

```bash
curl -s -X POST $E/api/ingest -H "$K" -H "Content-Type: application/json" \
  -d '{"workspace": "default:sales", "title": "Auth decision", "text": "Decided: refresh-token rotation ships in v2."}'
```

```json
{"ok": true, "signal_id": "...", "genre": "...", "type": "...", "entities": [], "source_package_id": "..."}
```

## 搜索

`GET /api/search?q=&workspace=&limit=&offset=&mode=&type=&as_of=`

```bash
curl -s "$E/api/search?q=refresh%20token&workspace=default:sales&limit=5" -H "$K"
```

```json
{
  "query": "refresh token",
  "results": [
    {
      "id": "...",
      "title": "Auth decision",
      "uri": "optimal://...",
      "content": "Decided: refresh-token rotation ships in v2.",
      "l0_abstract": "...",
      "score": 0.82,
      "workspace_id": "default:sales",
      "inherited": false
    }
  ],
  "pagination": {"offset": 0, "limit": 5, "total": 1, "has_more": false}
}
```

当没有嵌入模型回答时，`mode=hybrid`回答`503 hybrid_unavailable`。
错误的 `as_of` 答案 `400`。

| 路线 | 它有什么作用 |
|---|---|
| `GET /api/grep?q=` | 匹配的分块，带有 `intent`、`scale`、`modality`、`literal` 和 `path` 过滤器 |
| `GET /api/read?uri=&tier=` |一项由其 `optimal://` URI 表示；`tier` 可以是 `l0`、`l1` 或 `full` |
| `GET /api/recall/who`, `when`, `where`, `owns`, `actions` | 针对人物、时间、地点、所有权和操作的有目标召回 |
| `GET /api/profile` | 工作区画像：事实基准、近期活动和主要实体 |
| `GET /api/graph`, `/api/graph/hubs`, `/api/graph/clusters` |知识图谱|

## 询问并集合

`POST /api/rag` 为一个问题返回上下文；`GET /api/rag/stream` 以服务器发送事件的形式流式返回它。
`POST /api/assemble` 为智能体返回分层上下文包。
字段和响应请参阅[记忆与检索](/docs/optimal-engine/memory-and-retrieval#ask)。

```bash
curl -s -X POST $E/api/rag -H "$K" -H "Content-Type: application/json" \
  -d '{"query": "What did we decide about auth?", "workspace": "default:sales", "format": "markdown", "bandwidth": "small"}'

curl -N "$E/api/rag/stream?query=auth%20decision&workspace=default:sales" -H "$K"
```

## 内存

|路线 |它有什么作用 |
|---|---|
| `POST /api/memory` |存储内存：`{"content", "workspace", "is_static", "audience", "citation_uri", "metadata"}`。以 `201` 返回该记忆、其 `version` 和 `was_existing`。 |
| `POST /api/memory/remember` |通过门存放；答案`{"action", "memory", "pending_claim_id", "gate"}` |
| `GET /api/memory` |列表; `q`、`audience`、`include_forgotten`、分页 |
| `GET /api/memory/:id` |一份记忆|
| `GET /api/memory/:id/versions`, `/relations` |它的历史和联系|
| `POST /api/memory/:id/update`, `/extend`, `/derive` |新的链接版本 |
| `POST /api/memory/:id/forget` |隐藏它 |
| `DELETE /api/memory/:id` |删除它 |

## 索赔

|路线 |范围 |
|---|---|
| `GET /api/memory-core/claim-review` | `read` |
| `GET /api/memory-core/claims`, `/claims/:id` | `read` |
| `POST /api/memory-core/claims/:id/promote` | `claims:review` |
| `POST /api/memory-core/claims/:id/reject` | `claims:review` |

## 聊天会话

|路线 |它有什么作用 |
|---|---|
| `POST /api/sessions/commit` |存储聊天记录；请参阅[来源和摄取](/docs/optimal-engine/ingestion#chat-sessions) |
| `GET /api/sessions` |承诺聊天，最新的优先 |
| `GET /api/sessions/:id` |一聊及所学复习状态|

## 操作

这些需要一个 `admin` 键。

|路线 |它有什么作用 |
|---|---|
| `POST /api/auth/keys`, `GET /api/auth/keys` |铸造和列出密钥 |
| `POST /api/auth/keys/:id/revoke`、`DELETE /api/auth/keys/:id` |撤销或删除密钥 |
| `POST /api/backup` |立即备份 |
| `GET /api/backup/offsite` |异地备份 |
| `POST /api/workspaces/:id/bundle`, `POST /api/bundles/import` |导出或导入工作区包 |
| `GET /api/stores`, `GET /api/storage` | 存储、提供程序，以及对照大小配置的使用情况 |
| `/api/sync/*` |复制；请参阅[操作](/docs/optimal-engine/operations#replication) |

> **注意：应用程序数据**
>
> `/api/data` 是一个独立的、较小的应用程序记录 API：集合、记录、查询和更改流。
> 为它而设计的钥匙不能调用其他任何东西。
> 存储库指南 `docs/guides/app-data-api.md` 对此进行了描述。
