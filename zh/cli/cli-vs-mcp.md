---
slug: "cli/cli-vs-mcp"
title: "CLI 或 MCP：使用哪个"
description: "miosa CLI 适用于终端中的人类和脚本； MCP 为 AI 客户端提供了与工具相同的功能。两者都使用相同的 API 密钥和范围。"
section: "CLI, SDKs & MCP"
order: 11000
---

CLI 和 MCP

两种方式进入同一平台，具有相同的 API 密钥和相同的范围。选择您的客户可以运行的一个。

[CLI 概述](/docs/cli)[MCP 概述](/docs/mcp)[安装 CLI](/docs/cli/install)[MCP 工具](/docs/mcp/tools)

- **CLI** (`miosa`) 适用于人类、脚本和带有 shell 的智能体。您可以键入命令、读取文本或 JSON，然后根据退出代码进行分支。
- **MCP** 为 AI 客户端（Claude Code、Codex、Cursor、Claude Desktop）提供与工具相同的功能。该模型通过名称调用工具，而不是运行 shell 命令。

两者都使用**相同的 API 密钥和相同的范围**。可以从 CLI 创建沙箱的密钥可以通过 MCP 创建沙箱，反之亦然。唯一的区别是表面。

## 智能体应该使用哪个？

- **有壳吗？使用 CLI。** Claude Code、Codex、Cursor 的终端和任何 CI 作业都可以运行 `miosa`。 CLI 通常是更好的选择：它覆盖整个平台，而不仅仅是一个工具界面；输出可编写脚本；它由`jq`、管道和文件组成。
- **没有外壳？使用MCP。** Claude 桌面聊天和其他托管聊天产品没有外壳，因此工具是唯一的路径。 MCP 允许模型通过名称调用功能。

无论哪种方式都不会丢失任何内容，因为密钥、范围和资源是相同的。

## 比较

| |命令行 | MCP|
|---|---|---|
|谁驾驶它 |一个人、一个脚本或一个带有 shell 的智能体 |一款没有 shell 的 AI 客户端（Claude Desktop），或者更喜欢工具的 |
|接口 | `miosa <command> [flags]` |通过 Streamable HTTP 或 stdio 的 JSON-RPC 工具 |
|安装 | npm、Homebrew、安装脚本、发布 |托管端点，无需安装，或 `pip install miosa-mcp` |
|输出|文本或 JSON，带有退出代码 |结构化工具结果和图像 |
|覆盖范围|整个平台，加上任何端点的 `miosa api` | MCP工具表面|
|最适合 |具有 shell 访问权限、构建循环、CI、一次性操作的智能体 |无壳客户端 |
|授权 | `msk_u_*` 键或配置文件 | `msk_u_*` 键作为承载令牌 |
|范围模型 |相同的关键范围 |相同的关键范围 |

## 两者都使用

常见的拆分：带有 shell 的智能体使用 CLI 驱动平台，而没有 shell 的客户端使用相同的密钥通过 MCP 访问相同的帐户。

```bash
ID=$(miosa create agent-box --size small --wait --json | jq -r .id)   # CLI, from a shell agent
miosa exec "$ID" -- npm test
# the same account over MCP: exec(sandbox_id=ID, command="npm run build")
miosa list --json | jq -r '.data[].name'                              # CLI, read back
```

- [CLI 概述](/docs/cli)：安装、命令组、全局标志和退出代码。
- [MCP 概述](/docs/mcp)：连接 Claude Code、Codex、Cursor 或 Claude Desktop。

> **注意**
>
> 两个表面都继承相同的租户范围、计划限制、信用检查和审核日志记录，因为两者都位于 MIOSA REST API 前面。
