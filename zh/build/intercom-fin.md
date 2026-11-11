---
slug: "build/intercom-fin"
title: "建立 Fin 式支持智能体"
description: "Fin：解决客户问题的人工智能支持智能体。使用确切的资源、步骤和命令在 MIOSA 上构建它。"
section: "Build"
order: 10057
---

`~25 min` `TypeScript` `Python`

> **您正在构建的**：解决客户问题的人工智能支持智能体。
>
> **您将使用的原语**：沙箱、智能体和工具、Postgres、部署（Docker 主机）

## 你正在构建什么

一个 AI 支持智能体，可解决客户票证，以 Intercom 的 Fin 为模型。

Fin 通过帮助中心和公司数据回答客户问题，通过工具采取行动，并在无法解决时将问题移交给人工处理。

您将使用的基元：沙箱·智能体和工具·Postgres·部署（App Engine）

- 解决支持票证问题
- 来自帮助中心的答案
- 通过工具采取行动
- 交给人类

资料来源：[www.intercom.com/fin](https://www.intercom.com/fin)。 Fin 是 Intercom 的商标；本指南不隶属于他们，也不受他们认可。

## MIOSA 需要什么

每一行都是在开始之前创建的一件事。该数字与使用它的步骤相匹配。

1. 组织和 API 密钥 - 确定每次调用的范围；工作人员只需要一把工作区钥匙即可。 - `miosa api-key create app-key --preset agent` [文档](/docs/platform/api-keys)
2. 沙箱 - 智能体在其中编写代码和运行命令的隔离 Linux 工作区。 - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
3. Postgres - 托管关系数据，在机器和生产中作为 DATABASE_URL 注入。 - `miosa api POST /databases -d '{"name":"app-db","engine":"postgresql"}'` [文档](/docs/data/postgres)
4. 每个客户一个工作区 - 隔离每个客户的机器、部署和数据。 - `miosa workspace create customer-1` [文档](/docs/platform/white-label)
5. 品牌和白标签 - 您的名字和预览、部署和桌面上的 slug；顾客永远不会看到 MIOSA。 - `miosa org` [文档](/docs/platform/white-label)
6. 使用计量和计费 - 每个客户的使用情况，以及哪个帐户为新机器付费。 - `miosa org bill` [文档](/docs/platform/usage-and-billing)
7. 智能体和工具 - 将提示转化为工作：选择运行使用的工具和模型。 - `miosa agent harnesses` [文档](/docs/api-reference/agent-runs)
8. 部署 (App Engine) - 将应用程序发布到不可变的版本化 URL，并进行回滚。 - `miosa deploy create --from-sandbox app-box --name app --wait` [文档](/docs/docker-host/overview)

## 架构

```mermaid
sequenceDiagram
  participant U as User
  participant P as Your app
  participant A as Agent sandbox
  participant T as Systems and index
  participant H as Human reviewer

  U->>P: open a case
  P->>A: run the agent with scoped tools
  A->>T: read records and retrieve sources
  T-->>A: facts with their sources
  A-->>P: a proposal with citations
  P->>H: request approval when it matters
  H-->>P: approve or edit
  P->>T: apply the change, write the audit row
```

## 它如何映射到 MIOSA

执行域工作的智能体 -> 智能体在沙箱中运行 (`miosa prompt`, client.runs) 流事件，并使用您定义的可能调用的工具

域记录和历史 -> 托管 Postgres (`DATABASE_URL`) 用于记录、引用和审计跟踪

每个客户都是隔离的 -> 每个客户一个工作区 (`miosa workspace create`)

人工审核和升级 -> 智能体批准 (`miosa agent approvals`) 以及产品中的审核队列

## 数据和存储

Postgres 是一个带有仅附加审核表的记录系统；智能体决定的任何内容都不会仅存在于提示中。

保留每个存储字段的来源，以便人们可以检查它们。

## 授权和租赁

您的组织就是平台；客户永远不会获得 MIOSA 帐户，他们会登录您的产品。每个客户在入职时创建一个工作区，隔离他们的机器、运行和数据。

```bash
miosa workspace create intercom-fin-customer-1
```

标记每台机器并使用 `metadata` 中的客户 ID 运行，并且切勿跨客户查询。使用 GET /api/v1/usage 计量每个客户的使用情况，并使用 PUT /api/v1/bill-to (/docs/platform/usage-and-billing) 设置付款人。

## 智能体循环

运行框架：OSA 是 MIOSA 自己的运行框架，可与您连接的任何模型提供商配合使用，包括您自己的模型。

模型：人类（克劳德）。呼叫使用我自己的提供商密钥（`miosa connections add models`）； MIOSA 平台密钥从未被使用过。

会话：每个项目或对话一次聊天，因此智能体可以保留其上下文。沙箱上的第一个 `miosa prompt` 使用 `--new-chat` （打印聊天 ID）；之后的每个回合都会经过 `--chat <chat-id>`。 `--reuse chat` 每个聊天保留一台新计算机，以便文件保留。

流式传输：使用 `miosa run follow <run-id>` 或 client.runs.streamEvents(run.id) 跟踪运行，并使用 `miosa run steer` 和 `miosa run interrupt` 引导或停止运行。

智能体仅使用您定义的工具在沙箱内运行。它建议；您的代码强制执行限制（退款上限、允许的收件人），因此模型无法绕过这些限制。

## 步骤1. 创建域数据库。
   ```bash
   miosa api POST /databases -d '{"name":"intercom-fin-db","engine":"postgresql"}'
   ```
   ```bash
   miosa db connection intercom-fin-db
   ```
   检查：域表是否存在：客户、票证、消息、操作。
2. 创建智能体沙箱。
   ```bash
   miosa create intercom-fin-box --template agent-python --wait
   ```
   检查：沙箱正在运行并设置了 DATABASE_URL。
3. 将知识库（帮助中心、政策）加载到向量索引中，以便答案引用政策。
   ```bash
   miosa api POST /databases -d '{"name":"intercom-fin-knowledge","engine":"qdrant"}'
   ```
   检查：政策、文档或文章问题检索到正确的段落。
4. 将智能体可以在真实系统上调用的工具（查找订单、限额内退款、更改地址）定义为带有类型参数的窄函数。
   ```bash
   miosa prompt --sandbox intercom-fin-box --harness osa --model <anthropic-model-id> --chat <chat-id> "Resolve ticket <id> using only the defined tools and the cited policy; escalate if unsure"
   ```
   检查：智能体仅使用这些工具完成案件。
5. 将人员置于重要的环节：智能体人提出建议，人员批准任何不可逆转或高风险的事情，以及低可信度案例随上下文升级。
   ```bash
   miosa agent approvals
   ```
   检查：超过限额的退款、发送或提交等待批准。
6. 为每个智能体操作编写审核记录：谁、什么、输入、输出和运行 ID。
   ```bash
   miosa audit
   ```
   检查：任何决策都可以从审计跟踪中重建。
7. 发布智能体API和审核UI。
   ```bash
   miosa deploy create --from-sandbox intercom-fin-box --name intercom-fin --dir /workspace --port 3000 --run-command "npm start" --wait
   ```
   检查：public_url 服务于审核队列。

## 限制和成本

客户数据是个人数据：仅存储任务需要的内容、满足删除请求并保留审核跟踪。

将租户数据分开：每个客户一个工作区，以及按租户过滤的每个查询。

在每个案例的运行和上限工具调用上设置 `--max-time`。

原型：将其保留在默认大小的一台计算机上，跳过不需要的副本和自定义主机名，并在完成后删除所有内容。

## 验收检查

智能体仅使用定义的工具端到端地解决代表性案例。

一项高风险的行动正在等待人类的批准。

审计追踪重构了决策。

每个客户都被隔离在自己的工作空间中，并且根据他们的使用情况进行计量。

它创建的所有内容都可以被删除，而不会留下任何运行的内容。

## 后续步骤

- [智能体提示生成器选择产品形状并将简介交给您的智能体。](/docs/build)
- [运行编码智能体 在云中运行 Claude Code、Codex 或 Cursor，或在其上构建产品。](/docs/build/run-agents)
- [比较 MIOSA MIOSA 与其他沙箱和智能体平台的比较。](/docs/compare)
- [您可以构建什么 MIOSA 支持的每个产品模式。](/docs/use-cases)
