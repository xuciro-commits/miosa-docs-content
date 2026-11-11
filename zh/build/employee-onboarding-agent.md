---
slug: "build/employee-onboarding-agent"
title: "建立员工入职智能体"
description: "员工入职智能体：引导新员工度过第一周。使用确切的资源、步骤和命令在 MIOSA 上构建它。"
section: "Build"
order: 10067
---

`~25 min` `TypeScript` `Python`

> **你正在构建什么**：引导新员工度过他们的第一周。
>
> **您将使用的原语**：沙箱、Qdrant 向量、智能体和工具、部署（Docker 主机）

## 你正在构建什么

一个智能体带领新员工度过入职第一周，提供他们需要的东西并给出答案

它运行一个清单，在沙箱中提供存储库和访问权限，并通过引用回答公司文档中的问题。

您将使用的基元：沙箱·Qdrant向量·智能体和利用·部署（App Engine）

- 运行入职检查表
- 提供存储库和访问权限
- 来自公司文档的答案并附有引文
- 向经理报告进度

这是一种模式，而不是某种产品的复制品。从[相关的 MIOSA 指南](/docs/guides/internal-business-os)开始，构建具有上述功能的最小版本。

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
  participant M as Teammate
  participant O as One organization
  participant W as Team workspace
  participant S as Sandbox
  participant A as Agent run

  M->>O: sign in with a role
  O->>W: scope machines and data to the team
  M->>A: ask for something
  A->>S: work inside the team sandbox
  A-->>M: a result, a draft or a built tool
  M->>M: approve anything consequential
```

## 它如何映射到 MIOSA

隔离配置 -> 具有范围凭证的 MIOSA 沙箱

手册中的答案 -> 托管 Qdrant (`QDRANT_URL`)

每次雇用的清单状态 -> 托管 Postgres (`DATABASE_URL`)

组织成员资格 -> `miosa org invite`, `miosa member add`

## 数据和存储

将应用程序数据保存在具有行级所有者（成员 ID 或团队）的托管 Postgres 中，并为智能体操作保留仅附加审核表。

## 授权和租赁

您的组织就是平台；客户永远不会获得 MIOSA 帐户，他们会登录您的产品。每个客户在入职时创建一个工作区，隔离他们的机器、运行和数据。

```bash
miosa workspace create employee-onboarding-agent-customer-1
```

标记每台机器并使用 `metadata` 中的客户 ID 运行，并且切勿跨客户查询。使用 GET /api/v1/usage 计量每个客户的使用情况，并使用 PUT /api/v1/bill-to (/docs/platform/usage-and-billing) 设置付款人。

## 智能体循环

运行框架：OSA 是 MIOSA 自己的运行框架，可与您连接的任何模型提供商配合使用，包括您自己的模型。

模型：人类（克劳德）。呼叫使用我自己的提供商密钥（`miosa connections add models`）； MIOSA 平台密钥从未被使用过。

会话：每个项目或对话一次聊天，因此智能体可以保留其上下文。沙箱上的第一个 `miosa prompt` 使用 `--new-chat` （打印聊天 ID）；之后的每个回合都会经过 `--chat <chat-id>`。 `--reuse chat` 每个聊天保留一台新计算机，以便文件保留。

流式传输：使用 `miosa run follow <run-id>` 或 client.runs.streamEvents(run.id) 跟踪运行，并使用 `miosa run steer` 和 `miosa run interrupt` 引导或停止运行。

智能体充当成员：每次运行都在 `metadata` 中携带请求成员的 ID，使用仅限于该成员或团队的凭据，并且不能超出该成员可以执行的操作。

## 步骤1. 存储清单和每位员工的进度。
   ```bash
   miosa api POST /databases -d '{"name":"employee-onboarding-agent-onboarding","engine":"postgresql"}'
   ```
   检查：新员工有一份包含项目状态的清单。
2. 使用限定范围的凭证（存储库、帐户、访问权限）在沙箱中配置每个项目所需的内容，并记录每个授权。
   ```bash
   miosa create employee-onboarding-agent-box --template agent-node --wait
   ```
   检查：每项授权均被记录并可撤销。
3. 回答手册中的问题并附上引文。
   ```bash
   miosa api POST /databases -d '{"name":"employee-onboarding-agent-handbook","engine":"qdrant"}'
   ```
   ```bash
   miosa prompt --sandbox employee-onboarding-agent-box --harness osa --model <anthropic-model-id> --chat <chat-id> "Answer from the handbook chunks; cite them"
   ```
   检查：通过手册页面的链接回答政策问题。
4. 向经理报告进度并升级被阻止的项目。
   检查：经理可以看到已完成的工作以及遇到的困难。
5. 控制访问：成员和角色，以及对智能体所做操作的审计跟踪。
   ```bash
   miosa member add dana@employee-onboarding-agent.com --role member
   ```
   ```bash
   miosa audit
   ```
   检查：被删除的成员将立即失去访问权限。
6. 发布内部UI。
   ```bash
   miosa deploy create --from-sandbox employee-onboarding-agent-box --name employee-onboarding-agent --dir /workspace --port 3000 --run-command "npm start" --wait
   ```
   检查：成员可以打开它，其他人不能。

## 限制和成本

连接器是真正的工作：为每个源系统的身份验证、速率限制和数据形状预算时间。

设置 `idle_timeout_sec` 以便未使用的团队沙箱暂停。

默认情况下首选只读访问权限；授予每个任务写入。

原型：将其保留在默认大小的一台计算机上，跳过不需要的副本和自定义主机名，并在完成后删除所有内容。

## 验收检查

成员只能看到其团队的数据和工具。

每个智能体操作都可归因于审核日志中的成员。

如果没有指南所述的批准，任何后果都不会发生。

每个客户都被隔离在自己的工作空间中，并且根据他们的使用情况进行计量。

它创建的所有内容都可以被删除，而不会留下任何运行的内容。

## 后续步骤

- [智能体提示生成器选择产品形状并将简介交给您的智能体。](/docs/build)
- [运行编码智能体 在云中运行 Claude Code、Codex 或 Cursor，或在其上构建产品。](/docs/build/run-agents)
- [比较 MIOSA MIOSA 与其他沙箱和智能体平台的比较。](/docs/compare)
- [您可以构建什么 MIOSA 支持的每个产品模式。](/docs/use-cases)
