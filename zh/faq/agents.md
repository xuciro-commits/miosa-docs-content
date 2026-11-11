---
slug: "faq/agents"
title: "智能体商"
description: "支持的工具、创建和运行智能体、OSA、模型密钥以及智能体运行的位置。"
section: "Help"
order: 16004
---

关于智能体运行框架、模型密钥以及智能体运行位置的简短回答。

## 支持哪些智能体运行框架？

`claude-code`、`codex`、`osa` 和 `custom`，通过 `miosa agent new --harness` 设置。
Agent Run 提供程序还包括 `claude`、`pi` 和 `hermes`。
请参阅[智能体 CLI](/docs/cli/agents) 和[配置智能体](/docs/guides/configure-agent)。

## 如何创建和运行智能体？

```bash
miosa agent new reviewer --harness claude-code --instructions-file review.md
miosa agent run reviewer "review PR 42" --sandbox my-box
miosa run list                 # follow, stop
```

请参阅[智能体 CLI](/docs/cli/agents)。

## 什么是 OSA？

OSA 是 MIOSA 的开源智能体。
它安装在您的计算机上，作为 MCP 客户端运行，并在您不需要终端 UI 时使用 `osa run "..."` 无头运行。
请参阅 [MCP](/docs/mcp) 和 [CLI 参考 ](/docs/cli)。

## 智能体是否使用 MIOSA 平台密钥？

不会。智能体使用您自己的模型连接，而不是 MIOSA 平台密钥。
在首次运行之前创建智能体范围的密钥并添加模型连接。
请参阅[智能体 CLI](/docs/cli/agents)。

## 智能体在哪里运行？

智能体针对设备工作：沙箱工作人员、计算机或本地/BYOC 设备。
Docker Host 是托管目标，而不是智能体设备。
请参阅[智能体设备](/docs/guides/agent-devices)和[选择智能体设备](/docs/guides/agent-device-selection)。

## 我可以从我自己的智能体循环中驱动 MIOSA 吗？

是的。从您自己的后端调用原语，或从 [智能体开发套件 (ADK)](/docs/adk) 开始。
请参阅[集成 AI 智能体](/docs/guides/agent)。
