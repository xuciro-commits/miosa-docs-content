---
slug: "build/browser-use"
title: "在 MIOSA 上运行浏览器使用"
description: "浏览器使用：安装开源库并让智能体驱动真正的浏览器。在 MIOSA 沙箱或计算机上安装并运行它，然后为您的团队或客户包装它。"
section: "Build"
order: 10025
---

`~15 min` `TypeScript` `Python`

> **您正在构建的内容**：安装开源库并让智能体驱动真正的浏览器。
>
> **您将使用的原语**：计算机、沙盒

> **提示：运行他们的，或构建您自己的**
>
> 本指南在 MIOSA 上运行 **浏览器使用**。你不必使用它。
> 您可以单独在 MIOSA 上构建自己的浏览器智能体：通过 CDP 驱动沙箱](/docs/browser-use)内的[实时浏览器，或带有屏幕截图、点击和输入的完整[计算机](/docs/computer-use)。
> 您拥有循环、提示、模型和产品。
> 请参阅[构建浏览器和计算机使用智能体](/docs/build/chatgpt-agent)。

## 你正在构建什么

一个开源库，可让 AI 智能体驱动基于浏览器使用建模的真实浏览器。

你给图书馆一个任务和一个模型；它循环控制 Chromium 浏览器（页面、点击、表单、提取），直到任务完成。

您将使用的基元：计算机·沙箱

- 驱动真实浏览器的智能体
- 从页面中提取结构化数据
- 适用于您选择的型号
- 开源且可嵌入

您在 MIOSA 上安装并**运行真正的浏览器使用**。首先阅读官方文档：[docs.browser-use.com](https://docs.browser-use.com/)。下面的每个安装命令都来自他们；如果文档和本指南不一致，则文档获胜。 Browser Use 是开源 Python 库。

## MIOSA 需要什么

每一行都是在开始之前创建的一件事。该数字与使用它的步骤相匹配。

1. 组织和 API 密钥 - 确定每次调用的范围；工作人员只需要一把工作区钥匙即可。 - `miosa api-key create app-key --preset agent` [文档](/docs/platform/api-keys)
2. 计算机 - 用于浏览器、鼠标和屏幕截图工作的持久 Linux 桌面。 - `miosa computer create app-desktop` [文档](/docs/computers/overview)
3. 沙箱 - 智能体在其中编写代码和运行命令的隔离 Linux 工作区。 - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
4. 每个客户一个工作区 - 隔离每个客户的机器、部署和数据。 - `miosa workspace create customer-1` [文档](/docs/platform/white-label)
5. 品牌和白标签 - 您的名字和预览、部署和桌面上的 slug；顾客永远不会看到 MIOSA。 - `miosa org` [文档](/docs/platform/white-label)
6. 使用计量和计费 - 每个客户的使用情况，以及哪个帐户为新机器付费。 - `miosa org bill` [文档](/docs/platform/usage-and-billing)

## 架构

```mermaid
sequenceDiagram
  participant U as You
  participant M as MIOSA Computer
  participant P as Model provider
  participant D as MIOSA Deploy

  U->>M: install the tool from its official docs
  M->>P: calls with your own provider key
  U->>M: run a real task
  M-->>U: output, files
  U->>D: publish it for your team or your customers
```

## 它如何在 MIOSA 上运行

它运行的地方 -> MIOSA 计算机（带有浏览器的持久桌面）

状态和文件 -> 脚本和输出位于计算机上的 /workspace 中；浏览器配置文件保留在计算机上。

模型 -> 您自己的提供商密钥，而不是 MIOSA 平台密钥

您的客户 -> 各一个工作空间，彼此隔离

## 授权和租赁

您的组织就是平台；客户永远不会获得 MIOSA 帐户，他们会登录您的产品。每个客户在入职时创建一个工作区，隔离他们的机器、运行和数据。

```bash
miosa workspace create browser-use-customer-1
```

标记每台机器并使用 `metadata` 中的客户 ID 运行，并且切勿跨客户查询。使用 GET /api/v1/usage 计量每个客户的使用情况，并使用 PUT /api/v1/bill-to (/docs/platform/usage-and-billing) 设置付款人。

## 安装浏览器使用

按顺序在计算机（其终端）内运行它们。命令来自官方文档。

```text
curl -LsSf https://astral.sh/uv/install.sh | sh
```

```text
python3 --version   # Python >= 3.11 is required
```

```text
uv init --python 3.12
```

```text
uv add browser-use
```

检查：安装完成，没有错误；接下来的部分将配置并启动它。

## 配置

将您自己的模型密钥放入 `.env` 中（README 示例使用 OPENAI_API_KEY）。一台 MIOSA 计算机已经有桌面和 Chrome 可供智能体驱动；使用浏览器 仅当您选择时才使用云浏览器。

模型：人类（克劳德）。调用使用我自己的提供商密钥（ANTHROPIC_API_KEY）； MIOSA 平台密钥从未被使用过。

```bash
export ANTHROPIC_API_KEY="..."   # set it in the process environment, or the Secrets API; never write it into files you commit
```

## 运行它

启动它并证明它有效。

```text
uv run agent.py   # the README's agent.py with your task
```

检查：工具启动无错误并响应。

## 持久化和访问

脚本和输出位于计算机上的 /workspace 中；浏览器配置文件保留在计算机上。

检查：机器暂停并恢复后，工具再次启动，状态完好。

## 使其成为产品

每个客户工作区一台计算机；前面排着队；每个客户存储的结果和屏幕截图。

## 限制和成本固定您测试的版本并刻意更新；这些工具变化很快。

仅向该工具提供其所需的凭据（作为秘密），而不是写入文件中。

原型：将其保留在默认大小的一台计算机上，跳过不需要的副本和自定义主机名，并在完成后删除所有内容。

## 验收检查

Browser Use 在 MIOSA 计算机上运行并回答实际任务。

状态在暂停和恢复后仍然存在。

每个客户都有自己的工作区和自己的实例；他们之间没有任何东西是共享的。

它创建的所有内容都可以被删除，而不会留下任何运行的内容。

## 后续步骤

- [智能体提示构建器选择产品形状并将简介交给您的智能体。](/docs/build)
- [运行编码智能体 在云中运行 Claude Code、Codex 或 Cursor，或在其上构建产品。](/docs/build/run-agents)
- [比较 MIOSA MIOSA 与其他沙箱和智能体平台的比较。](/docs/compare)
- [您可以构建什么 MIOSA 支持的每个产品模式。](/docs/use-cases)
