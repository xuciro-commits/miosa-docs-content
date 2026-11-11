---
slug: "optimal-engine/miosa"
title: "MIOSA 上的Optimal Engine"
description: "在 MIOSA 上运行托管Optimal Engine - 控制台中的工作区内存、最佳聊天如何使用它、miosa 引擎命令、最佳 CLI 和 MCP 服务器的客户端密钥、API、大小、备份以及如何启用它。"
section: "Optimal Engine"
order: 8010
---

在 MIOSA 上，Optimal Engine作为托管数据库与引擎 `optimal` 一起运行。
MIOSA 为您创建并运行它：您无需安装它、持有其管理密钥或手动进行备份。

## 可用性

每个组织都启用托管引擎。
直到您的启用为止：

- 控制台在**数据库**下的创建流程不提供Optimal Engine；
- **启动内存**回答“内存无法启动。您的角色可能不允许，或者尚未为此组织设置内存。”；
- 通过 API 或 CLI 创建一个答案 `422`，代码为 `ENGINE_NOT_AVAILABLE`。

要启用它，请从控制台打开票证或发送电子邮件至 [support@miosa.ai](mailto:support@miosa.ai) 并附上您组织的名称。
请参阅[支持](/docs/faq/support)。
自托管无需启用；请参阅[快速入门](/docs/optimal-engine/quickstart)。

## 你得到什么

|部分|详情 |
|---|---|
|机器|您工作空间网络中的私人计算机，只能在 MIOSA 内部并通过 MIOSA API 访问 |
|数据|经得起重启和升级的耐用数据盘 |
|按键| MIOSA 持有引擎的管理密钥并且从不显示它；你铸造了短期的客户端密钥|
|工作空间 |以附加到它的每个 MIOSA 工作区命名的引擎工作区 |
|备份|全数据盘定时备份、按需备份|
|健康 |引擎本身报告的健康检查|
|搜索 |全文搜索。托管引擎不包含嵌入模型，因此矢量搜索在 MIOSA 上不可用。 |

它运行时按常开托管数据库的方式计费，价格为其规格对应的每小时单价。
控制台会在创建任何内容之前显示该价格以及整个月的价格。

## 控制台中的工作区内存

**Optimal > Memory**（`/optimal/memory`）是最简单的入口。
Memory 只属于一个工作区：请先在工作区切换器中选定一个工作区。

1. 按**启动内存**。
   - 如果您的组织已经有运行中的引擎，Memory 会把这个工作区附加到它上面，不会产生新的费用。
   - 否则控制台会提议创建一个 1 vCPU、2 GiB 内存、最小磁盘的引擎，并显示其价格。按**创建并启动内存**。
2. “内存正在启动。这大约需要一分钟。”
3. 运行时，该页面有一个搜索框，包含工作区已知的所有内容，以及每个结果的标题、位置和摘录。

要向 Memory 添加内容，请在聊天中让 Optimal 保存一条笔记。

## 最佳聊天如何使用内存

当工作区拥有 Memory 时，[Optimal](/docs/optimal) 可以：

- 在回答工作区的人员、项目、客户或历史相关问题之前先**搜索它**，并在 Memory 中没有任何内容时说明这一点；
- **保存**您要求其保留的注释、决定或记录；
- 当您询问时，**启动**工作空间的内存。

在 **All workspaces** 聊天中，Optimal 会搜索您可以打开的每个工作区，并注明每条结果来自哪个工作区。

每个聊天还有一个内存面板：

- **Save to memory** 会把该聊天保存为一个会话。
- **Sources** 列出 Optimal 读取过的内容，在组织聊天中按工作区分组。
- **在此聊天中了解到**列出了聊天产生的声明；接受或拒绝每一项。

Optimal 以您的身份、按您的权限通过 MIOSA 访问 Memory。
它不持有任何引擎密钥，也没有通往引擎的网络路径。
内存返回的内容作为不受信任的内容传递给模型。

## 从终端

[MIOSA CLI](/docs/cli) 有一个 `miosa engine` 命令组，版本 2.3.19 或更高版本。

```bash
miosa engine create brain --workspace notes --for workspace --wait
miosa engine list
miosa engine status notes
```

当存在具有该名称的引擎时，`create` 返回现有引擎，因此脚本每次都可以调用它。
`--for` 选择起始规格：`agent`（tiny，2 GiB 磁盘）、`workspace`（xs，10 GiB）或 `business`（small，25 GiB）。
`--size tiny|xs|small|medium|large` 和 `--disk <GiB>` 覆盖它。
为某个工作区创建的引擎会在首次运行时附加到该工作区。

|命令 |它有什么作用 |
|---|---|
| `miosa engine attach <workspace> --to <engine>` |将另一个工作区附加到引擎 |
| `miosa engine detach <workspace>` |分离工作区并撤销其引擎密钥 |
| `miosa engine status [workspace]` |哪个引擎为工作区提供服务，或者每个引擎及其工作区|
| `miosa engine connect <engine>` |创建客户端密钥并编写 `optimal` CLI 配置文件 |
| `miosa engine shell <engine> [-- command]` |使用退出时撤销的密钥运行 shell 或命令 |
| `miosa engine backup <engine>`, `miosa engine backups <engine>` |立即备份，或列出备份 |
| `miosa engine recover <engine>` |通过引擎恢复备份，或使用 `--to-new <name>` 恢复到新备份 |
| `miosa engine upgrade <engine> --to <version>`、`miosa engine rollback <engine>` |更改引擎版本 |
| `miosa engine rm <engine>` |删除引擎及其数据盘 |

### 使用最佳的 CLI 和 MCP 服务器

`miosa engine connect` 为引擎的工作区之一创建读/写密钥，并以模式 `0600` 写入 `~/.config/optimal/profile`。
[`optimal` CLI 和 MCP 服务器](/docs/optimal-engine/cli-sdks-mcp) 读取该配置文件，因此它们无需其他设置即可到达托管引擎。

```bash
miosa engine connect brain                       # key lasts 24 hours
miosa engine connect brain --workspace notes --profile notes --ttl 72h
optimal boot
optimal search "renewal"
miosa engine shell brain -- optimal brief        # one command, key revoked after
```

`--print` 打印 shell 导出而不是编写配置文件。
密钥默认有效期为 24 小时，最多 30 天。

## 工作区中的机器

附加到引擎的工作区中的沙箱、计算机、应用程序和智能体以两个变量开始：

|变量|价值|
|---|---|
| `OPTIMAL_ENGINE_URL` |引擎的私有地址，端口 4200 |
| `OPTIMAL_ENGINE_API_KEY` |仅限于该工作区的密钥 |

`optimal` CLI 和 MCP 服务器读取两者，因此计算机上的智能体可以立即调用 `optimal boot`。
当发动机停机时，机器仍然可以启动。

## 调用引擎API

从工作区外部，使用客户端密钥通过 MIOSA 调用引擎。

1. 创建客户端密钥。只有组织所有者或管理员才能创建一个。

   ```bash
   curl -s -X POST https://api.miosa.ai/api/v1/databases/$ENGINE_ID/client-keys \
     -H "Authorization: Bearer $MIOSA_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"workspace_id": "your_workspace_id", "ttl_seconds": 86400}'
   ```

   `201` 返回 `data.key` 一次，及其 `id` 和 `expires_at`。
   工作区必须附加到引擎。
   `GET /databases/:id/client-keys` 列出不带秘密的密钥，并且 `DELETE /databases/:id/client-keys/:key_id` 撤销一个密钥。

2. 用它调用引擎：

   ```bash
   curl -s "https://api.miosa.ai/api/v1/engines/$ENGINE_ID/engine-api/api/search?q=renewal" \
     -H "Authorization: Bearer $ENGINE_CLIENT_KEY"
   ```

智能体仅转发智能体所需的路由，并回答`404`其他所有路由，包括密钥、同步、备份和维护路由。

| 方法 | 位于 `/api/` 下的路由 |
|---|---|
| `GET` | `boot`、`brief`、`protocol`、`operating-projection`、`rhythm/current`、`loops`、`search`、`grep`、`read`、`memory`、 `memory-core/claims`、`memory-core/claim-review`、`sessions`、`sessions/:id`、`org-policy`、`review/routing`、`routing/rules`、`authority/effective`、`viability`、`viability/history`、 `analyses`、`jobs`、`workspaces`、`status`、`health`、`version`、`profile`、`recall/:kind`、`wiki/:slug`、`render/specs`、 `agent-actions/:id` |
| `POST` | `rag`、`assemble`、`reconstruct`、`ingest`、`memory`、`memory/remember`、`memory/:id/forget`、`loops`、`loops/extract`、`loops/:id/:action`、 `sessions/commit`、`routing/classify`、`signals/check`、`authority/decide`、`authority/requests`、`agent-actions`、`render/context` |

|状态 |代码|当 |
|---|---|---|
| 404 | `NOT_FOUND` | 路线不在列表中 |
| 409 | `ENGINE_NOT_RUNNING` | 发动机停止或启动 |
| 413 | `REQUEST_TOO_LARGE` | 正文超过 1 MiB，或查询字符串超过 8 KiB |
| 502 | `ENGINE_UNAVAILABLE`、`ENGINE_RESPONSE_TOO_LARGE` | 引擎没有回答，或者回答太多 |

### 工作空间内存路径

它们使用您的 MIOSA API 密钥（而不是客户端密钥）并到达为工作区提供服务的引擎。

所有路径均以 `https://api.miosa.ai/api/v1/workspaces/:workspace_id/engine` 开头。

|路线 |它有什么作用 |许可|
|---|---|---|
| `GET .../engine` | 哪个引擎为工作区提供服务 | |
| `GET .../engine/search?q=&limit=` |搜索工作区的内存| `databases:read` |
| `GET .../engine/boot` |工作区的启动简报 | `databases:read` |
| `POST .../engine/remember` 与 `{"content", "metadata"}` |保存记忆 | `databases:write` |
| `POST .../engine/sessions` 与 `{"correlation_id", "messages", "title"}` |将聊天保存为会话 | `databases:write` |
| `GET .../engine/sessions/:session_id` |阅读已保存的聊天记录 | `databases:read` |
| `POST .../engine/claims/:claim_id/accept` 或 `/reject` |审查索赔 | `databases:write` |

没有引擎答案的工作区 `404 NOT_FOUND`;未运行的引擎回答 `409 ENGINE_NOT_RUNNING`。

## 备份、升级和大小

在控制台中的 **Databases** 下打开引擎。
其页面有一个**概述**选项卡，其中包含针对大小限制的连接、运行状况和使用情况、**备份**选项卡、**调整大小**按钮，以及当引擎报告它们时，**简要**、**循环**、**工作空间**和**密钥**选项卡。

|尺寸| vCPU |内存|磁盘 |
|---|---|---|---|
| `agent` | 1 | 1 GiB | 2 GiB |
| `workspace` | 1 | 2 GiB | 10 GiB |
| `business` | 2 | 4GiB | 25 GiB |
| `platform` | 4 | 8 GiB | 100 GiB |
| `scale` | 8 | 16 GiB | 250 GiB |

在 `workspace`、`business`、`platform` 和 `scale` 之间移动调整大小； `agent` 只是起始尺寸。
调整大小会拒绝磁盘或内存无法容纳引擎已使用的大小，并重新启动正在运行的引擎。
通过 API，`POST /api/v1/databases/:id/resize` 接受 `{"size": "business"}`，并且 `"dry_run": true` 返回计划而不更改任何内容。

`miosa engine recover --to-new` 将备份恢复到新引擎中，并且永远不会触及原始引擎。
恢复引擎会保留之前的数据； `GET /api/v1/databases/:id/restore-copies` 列出了这些副本。

> **注意：自托管并一起管理**
>
> 托管引擎与自托管引擎是相同的软件，因此 [API](/docs/optimal-engine/api)、[CLI](/docs/optimal-engine/cli-sdks-mcp) 和 [概念](/docs/optimal-engine/concepts) 对两者都适用。
