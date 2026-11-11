---
slug: "optimal-engine/agents-and-workflows"
title: "智能体和工作流程"
description: "Optimal Engine为智能体提供的内容 - 启动简报、开环、每日节奏、操作协议、允许、询问或拒绝规则、组织策略、计划分析、网络钩子和操作跟踪。"
section: "Optimal Engine"
order: 8007
---

Optimal Engine 本身不运行智能体或调用工具。
它是智能体工作的记忆和规则：首先读取什么，打开什么，它可以做什么，以及它做了什么的记录。
任何智能体通过 HTTP、`optimal` CLI 或 MCP 服务器到达它；请参阅[CLI、SDK 和 MCP](/docs/optimal-engine/cli-sdks-mcp)。

## 会话开始：boot

```bash
optimal boot -w default:notes
```

`GET /api/boot?workspace=` 在一次调用中返回：

- 今天的日期和当前的节奏音符；
- 逾期或今天到期的循环；
- 最近的决定和过时的项目；
- 简介和操作规程；
- 工作区遵循的组织政策。

智能体应该在会话中的任何其他操作之前调用它。

## 日常操作

|路线 |命令行 |它有什么作用 |
|---|---|---|
| `GET /api/brief` | `optimal brief` |今天需要做出决定的排名项目 |
| `GET /api/loops`、`POST /api/loops` | `optimal loops`、`optimal loops add` |开环：承诺、应回复、后续行动、付款、所需决策 |
| `POST /api/loops/extract` | `optimal loops extract` |在最近的来源中寻找新的循环 |
| `POST /api/loops/:id/close`、`drop`、`reopen` | `optimal loops close` |更改循环的状态 |
| `GET /api/rhythm/current` | `optimal today` |每日、每周或每月的笔记 |
| `GET /api/protocol`, `PUT /api/protocol` | `optimal protocol` |智能体在此工作区中遵循的协议 |

## 智能体可以做什么的规则

### 权威

权威答案允许、请求或拒绝对资源的操作，例如读取、写入、导出、擦除或同步。
否认就是胜利，没有规则的事情都是被否认的。

```bash
optimal authority decide --action export --resource "optimal://default:notes/..." -w default:notes
```

|路线 |它有什么作用 |
|---|---|
| `POST /api/authority/decide` |寻求决定 |
| `GET /api/authority/effective` |适用于某个主题的规则 |
| `POST /api/authority/requests` |请求某人批准某项行动 |
| `/api/authority/policies`、`/api/authority/grants` |管理规则和限时补助金 |

使用 `OPTIMAL_AUTHORITY_MODE` 打开它：默认情况下 `off`，`observe` 记录决策而不阻塞，或 `enforce`。

### 组织政策

`PUT /api/org-section` 为组织设置一次规则。
每个工作区通过 `GET /api/org-policy` 读取它们，无法更改它们，并在 `boot` 和 `assemble` 中接收它们。

### 预发送支票

`POST /api/signals/check` (`optimal check`) 根据接收者和通道检查草稿消息，并建议较短的版本。

## 工作记录

|路线 |它有什么作用 |
|---|---|
| `POST /api/agent-actions`、`GET /api/agent-actions/:id` |记录特工的行为，并通过相关证据阅读它 |
| `POST /api/sessions/commit` |将完成的聊天存储为一个片段；所学到的知识将用于索赔审查|
| `GET /api/viability` |作为操作系统的工作区健康报告，带有警报 |

## 预定的分析和作业

引擎按计划运行六次分析：`promises`、`workspace_population`、`meeting_archive`、`file_audit`、`storage_usage` 和 `retention_sweep`。

|路线 |它有什么作用 |
|---|---|
| `GET /api/analyses` |他们的报告|
| `POST /api/analyses/run` |现在运行一个 (`admin`) |
| `GET /api/jobs` |作业队列 |
| `GET /api/jobs/dead`、`POST /api/jobs/dead/:id/replay` |作业失败并重试 |

## Webhook 和实时更新

订阅说明了智能体或个人想要推送给他们的内容：工作区中的任何内容、一个节点、一个主题或一个受众。
使用 `POST /api/subscriptions` 创建一个；它可以暂停、恢复和删除。
当订阅的 `metadata` 具有 `webhook_url` 时，引擎会将匹配项推送到 `GET /api/surface/stream`（服务器发送的事件流）并推送到 Webhook。
`POST /api/subscriptions/:id/test-webhook` 发送测试交付。

> **注意：工作流程和技能包**
>
> 引擎记录工作流程轨迹，并可以将重复的轨迹提升到技能包中（`mix optimal.skills`）。
> 这些是供审查和检索的记录。
> 引擎不会将技能包作为代码运行，并且它没有智能体注册表或智能体之间的委派。
