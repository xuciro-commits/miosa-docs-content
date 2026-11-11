---
slug: "build/phind"
title: "构建 Phind 风格的答案引擎"
description: "Phind：以交互式迷你应用程序呈现的技术答案。使用确切的资源、步骤和命令在 MIOSA 上构建它。"
section: "Build"
order: 10037
---

`~25 min` `TypeScript` `Python`

> **您正在构建什么**：以交互式迷你应用程序呈现的技术答案。
>
> **您将使用的原语**：沙箱、智能体和利用、Qdrant 向量、部署（Docker 主机）

## 你正在构建什么

一个技术答案引擎，将其答案呈现为一个小型交互式应用程序，而不是由 Phind 在 Phind 上建模的文本墙。

开发人员问题会触发文档和网络搜索；引擎返回的不是散文，而是一个渲染的迷你应用程序（一个工作代码示例、一个计算器、一个图表）来演示答案。

您将使用的基元：沙箱·智能体和利用·Qdrant向量·部署（App Engine）

- 技术问题解答
- 答案呈现为可运行的迷你应用程序
- 您可以运行的代码示例
- 引用它使用的文档

来源：[phind.com](https://phind.com/)。 Phind 是其所有者的商标；本指南不隶属于他们，也不受他们认可。

## MIOSA 需要什么

每一行都是在开始之前创建的一件事。该数字与使用它的步骤相匹配。

1. 组织和 API 密钥 - 确定每次调用的范围；工作人员只需要一把工作区钥匙即可。 - `miosa api-key create app-key --preset agent` [文档](/docs/platform/api-keys)
2. 沙箱 - 智能体在其中编写代码和运行命令的隔离 Linux 工作区。 - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
3. Qdrant 矢量 - 用于搜索和嵌入的托管矢量存储，作为 QDRANT_URL 注入。 - `miosa api POST /databases -d '{"name":"app-index","engine":"qdrant"}'` [文档](/docs/data/qdrant)
4. 每个客户一个工作区 - 隔离每个客户的机器、部署和数据。 - `miosa workspace create customer-1` [文档](/docs/platform/white-label)
5. 品牌和白标签 - 您的名字和预览、部署和桌面上的 slug；顾客永远不会看到 MIOSA。 - `miosa org` [文档](/docs/platform/white-label)
6. 使用计量和计费 - 每个客户的使用情况，以及哪个帐户为新机器付费。 - `miosa org bill` [文档](/docs/platform/usage-and-billing)
7. 智能体和工具 - 将提示转化为工作：选择运行使用的工具和模型。 - `miosa agent harnesses` [文档](/docs/api-reference/agent-runs)
8. 部署 (App Engine) - 将应用程序发布到不可变的版本化 URL，并进行回滚。 - `miosa deploy create --from-sandbox app-box --name app --wait` [文档](/docs/docker-host/overview)

## 架构

```mermaid
sequenceDiagram
  participant U as User
  participant P as Your pipeline
  participant S as Sandbox
  participant V as Vector index
  participant M as Model

  U->>P: ask a question
  P->>S: fetch and extract the top pages
  S->>V: embed and store chunks
  P->>V: retrieve the best chunks
  P->>M: answer from those chunks only
  M-->>P: answer with citations
  P-->>U: answer and its sources
```

## 它如何映射到 MIOSA

搜索并读取源 -> 运行获取/提取作业的 MIOSA 沙箱（当页面需要真正的浏览器时需要一台计算机）

检索其读取的内容 -> 用于块嵌入的托管 Qdrant (`QDRANT_URL`)

带引用的答案 -> 智能体运行 (`miosa prompt`, client.runs) 流事件，仅从检索到的块中应答并返回源 ID

提供答案 UI 或 API -> MIOSA 部署 (`miosa deploy create`)，不可变且版本化

## 数据和存储

Qdrant 保存块嵌入；有效负载携带 url、标题、块 id 和 fetched_at，因此每个答案都可以引用已保存的页面。

将话题和答案及其引用保存在您自己的数据存储中。

## 授权和租赁

您的组织就是平台；客户永远不会获得 MIOSA 帐户，他们会登录您的产品。每个客户在入职时创建一个工作区，隔离他们的机器、运行和数据。

```bash
miosa workspace create phind-customer-1
```

标记每台机器并使用 `metadata` 中的客户 ID 运行，并且切勿跨客户查询。使用 GET /api/v1/usage 计量每个客户的使用情况，并使用 PUT /api/v1/bill-to (/docs/platform/usage-and-billing) 设置付款人。

## 智能体循环

运行框架：OSA 是 MIOSA 自己的运行框架，可与您连接的任何模型提供商配合使用，包括您自己的模型。

模型：人类（克劳德）。呼叫使用我自己的提供商密钥（`miosa connections add models`）； MIOSA 平台密钥从未被使用过。

会话：每个项目或对话一次聊天，因此智能体可以保留其上下文。沙箱上的第一个 `miosa prompt` 使用 `--new-chat` （打印聊天 ID）；之后的每个回合都会经过 `--chat <chat-id>`。 `--reuse chat` 每个聊天保留一台新计算机，以便文件保留。

流式传输：使用 `miosa run follow <run-id>` 或 client.runs.streamEvents(run.id) 跟踪运行，并使用 `miosa run steer` 和 `miosa run interrupt` 引导或停止运行。

检索第一，生成第二：模型只看到检索到的块，并且必须引用它们。该工具在管道沙箱中运行，因此获取和提取代码是隔离的。

## 步骤1. 创建向量索引。
   ```bash
   miosa api POST /databases -d '{"name":"phind-index","engine":"qdrant"}'
   ```
   检查：QDRANT_URL 和 QDRANT_API_KEY 可用于管道。
2. 创建用于获取、提取和索引页面的管道沙箱。
   ```bash
   miosa create phind-box --template agent-python --wait
   ```
   检查：沙箱正在运行并且可以获取公共 URL。
3. 检索：对于问题，调用您自带的搜索源（搜索 API）或抓取定义的语料库，获取页面，并提取带有 url、标题和 fetched_at 的干净文本。
   检查：每个存储的页面都有一个 URL 和一个获取时间。
4. 使用有效负载中的 url、标题和块 ID 进行块、嵌入和更新插入到 Qdrant 中。
   检查：Qdrant 中的块计数与您提取的内容相符。
5. 作为工件回答：仅向模型提供检索到的块，并要求提供一个小型独立应用程序（图表、计算器、可运行的代码示例）以及单行摘要，引用块编号。不要要求论文。
   ```bash
   miosa prompt --sandbox phind-box --harness osa --model <anthropic-model-id> --chat <chat-id> "Answer the question using only the provided chunks; cite them as [n]"
   ```
   检查：每个声明都带有解析为已保存页面的引用；无法回答的问题会变成“我找不到这个”。
6. 渲染工件：沙箱构建返回的应用程序，并且您的 UI 将其嵌入到源旁边的沙箱 iframe 中，因此答案是读者可以运行的内容，而不仅仅是阅读。
   ```bash
   miosa preview create phind-box 3000 --name answer
   ```
   检查：答案呈现为实时工件，并且从存储的规范中重新运行相同的问题会呈现相同的结果。
7. Follow-ups：保留线程id和最后的轮数，并在检索之前将follow-up重写为独立查询。
   检查：“那么它的价格呢？”检索正确的产品。
8. 发布答案 UI：首先流式传输答案文本，然后流式传输引文。
   ```bash
   miosa deploy create --from-sandbox phind-box --name phind --dir /workspace --port 3000 --run-command "npm start" --wait
   ```
   检查：public_url 端到端地回答了一个问题。

## 限制和成本

抓取时遵守 robots.txt 和速率限制；缓存获取的页面，因此重复的问题不会重新获取。

绑定检索：每个答案固定数量的块可以保持延迟和成本可预测。

原型：将其保留在默认大小的一台计算机上，跳过不需要的副本和自定义主机名，并在完成后删除所有内容。

## 验收检查

答案引用来源，引文解析为保存的页面。

指数无法回答的问题是被拒绝的，而不是发明的。

每个客户都被隔离在自己的工作空间中，并且根据他们的使用情况进行计量。

它创建的所有内容都可以被删除，而不会留下任何运行的内容。

## 后续步骤

- [智能体提示构建器选择产品形状并将简介交给您的智能体。](/docs/build)
- [运行编码智能体 在云中运行 Claude Code、Codex 或 Cursor，或在其上构建产品。](/docs/build/run-agents)
- [比较 MIOSA MIOSA 与其他沙箱和智能体平台的比较。](/docs/compare)
- [您可以构建什么 MIOSA 支持的每个产品模式。](/docs/use-cases)
