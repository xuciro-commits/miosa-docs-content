---
slug: "mcp"
title: "MCP概述"
description: "将 OSA、Claude Code、Codex、Cursor、Hermes 或 Claude Desktop 连接到 MIOSA MCP 服务器，以实现跨平台的沙箱、计算机、部署、存储和 151 种工具。"
section: "CLI, SDKs & MCP"
order: 11022
---

CLI 和 MCP

为任何 AI 客户提供完整的 MIOSA 工具：OSA、Claude Code、Codex、Cursor、Hermes。

`{ "mcpServers": { "miosa": { "url": "https://api.miosa.ai/api/v1/mcp", "headers": { "Authorization": "Bearer msk_u_..." } } } }`

OSA（我们的开源智能体）内的每个 MIOSA 工具都是 mcp__miosa__* 工具。

另存为 ~/.osa/mcp.json，或在 OSA 内运行 /mcp add。

- [安装和验证](/docs/mcp/install)：托管 HTTP 或本地 stdio、密钥和组织上下文。
- [从智能体使用](/docs/mcp#osa)：连接 OSA、Claude Code、Codex、Cursor 或 Hermes。
- [工具参考](/docs/mcp/tools)：所有 151 个工具，按系列分组。
- [CLI 或 MCP](/docs/cli/cli-vs-mcp)：向智能体提供哪一个，以及为什么两者都有效。

MIOSA 提供公共 **模型上下文协议** 服务器。使用 `msk_u_*` API 密钥将任何支持 MCP 的客户端指向它，您就可以获得 MIOSA 工具带。 [OSA](/docs/mcp#osa)，MIOSA 的开源智能体，是主要客户端，Claude Code、Codex、Cursor 和 Hermes 的工作方式相同。工具带：沙箱、完整的 GUI 计算机、快照、部署、数据库、存储、卷、cron、函数、webhooks、API 密钥、工作区、计算机服务和区域。

该页面是您自己的智能体调用的服务器。对于另一个方向，您添加到智能体的官方工具服务器，请参阅[工具服务器目录](/docs/mcp/catalog)。

托管端点：

```text
https://api.miosa.ai/api/v1/mcp
```

服务器使用 MCP 2025-03-26 Streamable HTTP、single-POST JSON-RPC。不需要 SSE 监听流。

## OSA

OSA 是 MIOSA 的开源智能体。它在您的计算机或 MIOSA 计算机上运行，​​并与您的 Claude 或 ChatGPT 订阅配合使用。它是一个MCP客户端，因此它将MIOSA服务器作为其内置工具之外的另一套工具；它们显示为 `mcp__miosa__*`。

安装 OSA（macOS、Linux 和 Windows），然后启动一次以运行设置向导：

```bash
curl -fsSL https://raw.githubusercontent.com/Miosa-osa/OSA/main/scripts/install.sh | sh
osa
```

在 Windows 上使用 `irm https://raw.githubusercontent.com/Miosa-osa/OSA/main/scripts/install.ps1 | iex`。 Homebrew 安装 `osagent` 二进制文件：`brew install Miosa-osa/homebrew-tap/osagent`。

将 MIOSA 添加到 `~/.osa/mcp.json`（用户范围）。该文件使用与 Claude Desktop 相同的 `mcpServers` 形状，并且 `url` 选择 HTTP 传输：

```json
{
  "mcpServers": {
    "miosa": {
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer msk_u_..." }
    }
  }
}
```

或者使用斜杠命令从 OSA 内部添加它，该命令写入相同的文件。 OSA 不会扩展 `headers` 中的环境变量，因此将键本身放入值中：

```text
/mcp add miosa https://api.miosa.ai/api/v1/mcp -t http -s user -H 'Authorization: Bearer msk_u_...'
```

运行 `/mcp` 列出服务器及其连接状态。项目 `.mcp.json` 也可以工作； OSA 在启动项目范围的服务器之前会请求批准。

## 克劳德代码

```bash
claude mcp add --transport http miosa \
  https://api.miosa.ai/api/v1/mcp \
  --header "Authorization: Bearer msk_u_..."
```

添加 `--scope user` 以使其在所有项目中可用。使用 `claude mcp list` 进行验证。

同样，在 `.mcp.json`（项目）或 `~/.claude.json`（用户）中：

```json
{
  "mcpServers": {
    "miosa": {
      "type": "http",
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer msk_u_..." }
    }
  }
}
```

## 法典

`~/.codex/config.toml`：

```toml
[mcp_servers.miosa]
url = "https://api.miosa.ai/api/v1/mcp"
bearer_token_env_var = "MIOSA_API_KEY"
```

`bearer_token_env_var` 命名保存密钥的环境变量，而不是密钥本身。该变量必须存在于启动 Codex 的环境中。一行形式：

```bash
codex mcp add miosa --url https://api.miosa.ai/api/v1/mcp --bearer-token-env-var=MIOSA_API_KEY
```

## 光标

`~/.cursor/mcp.json`（全局）或 `.cursor/mcp.json`（每个项目）：

```json
{
  "mcpServers": {
    "miosa": {
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer msk_u_..." }
    }
  }
}
```

## 爱马仕

[Hermes Agent](https://github.com/NousResearch/hermes-agent) 是 Nous Research 的开源智能体。 MCP 支持随标准安装一起提供。将 MIOSA 添加到 `~/.hermes/config.yaml`：

```yaml
mcp_servers:
  miosa:
    url: "https://api.miosa.ai/api/v1/mcp"
    headers:
      Authorization: "Bearer ${MIOSA_API_KEY}"
```

Hermes 在连接时从环境和 `~/.hermes/.env` 解析 `url` 和 `headers` 中的 `${MIOSA_API_KEY}`，因此密钥不会出现在配置文件中。将 `MIOSA_API_KEY=msk_u_...` 放入 `~/.hermes/.env`，然后启动 `hermes chat`。

## 克劳德桌面

Claude Desktop 的 `claude_desktop_config.json` 仅是 stdio。使用 `mcp-remote` 桥，它作为 stdio 服务器启动并智能体托管端点。

`~/Library/Application Support/Claude/claude_desktop_config.json`：

```json
{
  "mcpServers": {
    "miosa": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.miosa.ai/api/v1/mcp",
        "--header",
        "Authorization: Bearer ${MIOSA_API_KEY}"
      ],
      "env": { "MIOSA_API_KEY": "msk_u_..." }
    }
  }
}
```

编辑后重新启动应用程序（不仅仅是重新加载）。或者，在“设置”、“连接器”、“添加自定义连接器”下添加托管 URL，并在其中设置授权标头。

> **注意**
>
> 当您需要使用自定义本地逻辑包装 MCP 层，或者您的客户端不支持远程 MCP 时，首选本地 stdio 服务器（Python 包）。两种模式都使用相同的 MIOSA REST API。请参阅[安装并验证](/docs/mcp/install)。

## 操作权限

受控 MCP 工具在调度前立即询问 MIOSA 的控制平面行动权限。控制平面拥有能力版本、授权、一次性批准、撤销和收据。 MCP 服务器仅将工具名称映射到生成的合约中的规范功能，并且从不在本地复制策略。每个注册的工具都具有一种规范功能。没有映射的工具在任何调度之前都无法关闭。在 SDK 或 REST 操作运行之前，等待批准、拒绝、缺少目录功能或不可用的权限都会失败关闭。

## 活动计算机

所有计算机工具都接受可选的 `computer_id`。省略时，服务器将使用最近创建或访问的计算机，因此您可以 `computer_create` 一次并在会话的其余部分省略 ID。

## 错误

失败的工具调用使用 MCP `isError=true` 并包含一个结构化错误对象，该对象保留后端错误代码、HTTP 状态、请求 ID、响应详细信息、工具名称和人类可读的消息。客户端可以在稳定的代码上进行分支，例如 `INVALID_TENANT_CONTEXT`，而无需解析显示文本。

- [安装并验证](/docs/mcp/install)：托管 HTTP 和本地 stdio、密钥和租户上下文。
- [工具参考](/docs/mcp/tools)：所有 151 个工具，按系列分组。
