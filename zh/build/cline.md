---
slug: "build/cline"
title: "在 MIOSA 上运行 Cline"
description: "Cline：安装最多的开源编码智能体，在沙箱中无头。在 MIOSA 沙箱或计算机上安装并运行它，然后为您的团队或客户包装它。"
section: "Build"
order: 10017
---

`~15 min` `TypeScript` `Python`

> **您正在构建的**：安装最多的开源编码智能体，在沙箱中无头。
>
> **您将使用的基元**：沙箱

## 你正在构建什么

一个开源自主编码智能体（VS Code 扩展、JetBrains 插件、CLI 和 SDK），可编辑文件并运行具有可审查差异的命令，由 Cline Bot, Inc. 在 Cline 上建模。

Cline 循环工作：它读取存储库、提出计划、编辑文件并运行终端命令，显示可审查的差异和每个更改的检查点；计划和行动模式控制它在提出要求之前要做的事情。

您将使用的基元：沙箱

- 具有可审查差异的自主多文件编辑
- 运行终端命令并对它们的输出做出反应
- 具有每次操作批准的计划和行动模式
- 与模型无关：带上您自己的提供商密钥
- 无头 CLI，为您的产品提供 JSON 输出

您在 MIOSA 上安装并**运行真正的 Cline**。首先阅读官方文档：[docs.cline.bot/](https://docs.cline.bot/)。下面的每个安装命令都来自他们；如果文档和本指南不一致，则文档获胜。 Cline 是开源的（VS Code 扩展、JetBrains 插件、CLI 和 SDK）。

## MIOSA 需要什么

每一行都是在开始之前创建的一件事。该数字与使用它的步骤相匹配。

1. 组织和 API 密钥 - 确定每次调用的范围；工作人员只需要一把工作区钥匙即可。 - `miosa api-key create app-key --preset agent` [文档](/docs/platform/api-keys)
2. 沙箱 - 智能体在其中编写代码和运行命令的隔离 Linux 工作区。 - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
3. 每个客户一个工作区 - 隔离每个客户的机器、部署和数据。 - `miosa workspace create customer-1` [文档](/docs/platform/white-label)
4. 品牌和白标 - 您的姓名和预览、部署和桌面上的 slug；顾客永远不会看到 MIOSA。 - `miosa org` [文档](/docs/platform/white-label)
5. 使用计量和计费 - 每个客户的使用情况，以及哪个帐户为新机器付费。 - `miosa org bill` [文档](/docs/platform/usage-and-billing)

## 架构

```mermaid
sequenceDiagram
  participant U as You
  participant M as MIOSA Sandbox
  participant P as Model provider
  participant D as MIOSA Deploy

  U->>M: install the tool from its official docs
  M->>P: calls with your own provider key
  U->>M: run a real task
  M-->>U: output, files
  U->>D: publish it for your team or your customers
```

## 它如何在 MIOSA 上运行

它运行的地方 -> MIOSA 沙箱（一个独立的 Linux 工作区）

状态和文件 -> 将存储库保留在 /workspace 中；持久沙箱会在暂停期间保留它和 `~/.cline` 配置。

模型 -> 您自己的提供商密钥，而不是 MIOSA 平台密钥

您的客户 -> 各一个工作空间，彼此隔离

## 授权和租赁

您的组织就是平台；客户永远不会获得 MIOSA 帐户，他们会登录您的产品。每个客户在入职时创建一个工作区，隔离他们的机器、运行和数据。

```bash
miosa workspace create cline-customer-1
```

标记每台机器并使用 `metadata` 中的客户 ID 运行，并且切勿跨客户查询。使用 GET /api/v1/usage 计量每个客户的使用情况，并使用 PUT /api/v1/bill-to (/docs/platform/usage-and-billing) 设置付款人。

## 安装克莱因

按顺序在沙箱内运行它们（例如 `miosa exec cline-task-1 -- bash -lc '<command>'`）。命令来自官方文档。

```bash
npm i -g cline
```

检查：安装完成，没有错误；接下来的部分将配置并启动它。

## 配置

使用您自己的提供商密钥进行身份验证：运行 `cline auth` 并选择您已连接的提供商（Anthropic、OpenAI、Gemini、OpenRouter、Bedrock 或本地模型）。切勿使用 MIOSA 平台密钥。

模型：人类（克劳德）。调用使用我自己的提供商密钥（ANTHROPIC_API_KEY）； MIOSA 平台密钥从未被使用过。

```bash
export ANTHROPIC_API_KEY="..."   # set it in the process environment, or the Secrets API; never write it into files you commit
```

## 运行它

启动它并证明它有效。

```text
cline --version
```

```text
cline "summarise this repository and list the failing tests"
```

脚本化、非交互式使用（产品或队列的调用方式）：

```text
cline --json "summarise this repository and list the failing tests"
```

```text
cline --auto-approve true "run the tests and fix the failures"   # unattended: run it on a clean branch
```

检查：无头任务完成并打印结果。

## 持久化和访问

将存储库保存在 /workspace 中；持久沙箱会在暂停期间保留它和 `~/.cline` 配置。

检查：机器暂停并恢复后，工具再次启动，状态完好。

## 使其成为产品

在客户的工作区中运行 Cline 每个客户任务（将其安装在计算机中并使用 `miosa exec` 调用它）并将其 JSON 事件流式传输到您自己的 UI 中。

## 限制和成本

固定您测试的版本并刻意更新；这些工具变化很快。

仅向该工具提供其所需的凭据（作为秘密），而不是写入文件中。原型：将其保留在默认大小的一台计算机上，跳过不需要的副本和自定义主机名，并在完成后删除所有内容。

## 验收检查

Cline 在 MIOSA 沙箱上运行并回答实际任务。

状态在暂停和恢复后仍然存在。

每个客户都有自己的工作区和自己的实例；他们之间没有任何东西是共享的。

它创建的所有内容都可以被删除，而不会留下任何运行的内容。

## 后续步骤

- [智能体提示构建器选择产品形状并将简介交给您的智能体。](/docs/build)
- [运行编码智能体 在云中运行 Claude Code、Codex 或 Cursor，或在其上构建产品。](/docs/build/run-agents)
- [比较 MIOSA MIOSA 与其他沙箱和智能体平台的比较。](/docs/compare)
- [您可以构建什么 MIOSA 支持的每个产品模式。](/docs/use-cases)
