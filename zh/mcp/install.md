---
slug: "mcp/install"
title: "安装并验证"
description: "选择托管的 MIOSA MCP 端点或本地 stdio 包，创建密钥并设置组织上下文。"
section: "CLI, SDKs & MCP"
order: 11023
---

有两种方法可以运行 MIOSA MCP 服务器。两者都使用相同的 MIOSA REST API。唯一的区别是 JSON-RPC 桥是在 MIOSA 的基础设施（托管）中运行还是在您的计算机（stdio）上运行。

## 托管（推荐）

无需安装。使用您的密钥作为承载令牌将任何 MCP 客户端指向公共端点。

```text
https://api.miosa.ai/api/v1/mcp
```

```bash
claude mcp add --transport http miosa \
  https://api.miosa.ai/api/v1/mcp \
  --header "Authorization: Bearer msk_u_your_key_here"
```

有关 Claude Code、Codex、Cursor 和 Claude Desktop 的配置文件形式，请参阅[概述](/docs/mcp)。

## 本地 stdio

当您需要使用自定义本地逻辑包装 MCP 层，或者您的客户端不支持远程 MCP 时，请使用 stdio。

```bash
pip install miosa-mcp
```

需要 Python 3.10 或更高版本。软件包版本 0.4.3。

独立包注册自己的工具集和自己的名称：194 个具有前缀优先方案的工具（`sandbox_create`、`sandbox_exec`、`computer_create`）。它是与托管端点分开的服务器，它使用不同的方案注册 151 个工具（`create_sandbox`、`exec`、`computer_create`）。 [工具参考](/docs/mcp/tools)记录了托管服务器；本地包的工具名称与其自己的 `list_tools` 输出遵循相同的模式。

添加到`~/.claude/mcp.json`：

```json
{
  "mcpServers": {
    "miosa": {
      "command": "python",
      "args": ["-m", "miosa_mcp"],
      "env": {
        "MIOSA_API_KEY": "msk_u_your_key_here",
        "MIOSA_TENANT": "optional-organization-slug"
      }
    }
  }
}
```

## 连接 OSA

OSA 是 MIOSA 的开源智能体。它在您的计算机或 MIOSA 计算机上运行，​​并与您的 Claude 或 ChatGPT 订阅配合使用。它从 `~/.osa/mcp.json` 读取 MCP 服务器，因此托管端点是 `url` 加上 `Authorization` 标头。完整的设置，包括 `/mcp add` 命令，位于 [概述 ](/docs/mcp#osa) 中。

```json
{
  "mcpServers": {
    "miosa": {
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer msk_u_your_key_here" }
    }
  }
}
```

## 连接赫尔墨斯

Hermes Agent 从 `~/.hermes/config.yaml` 读取 `mcp_servers` 并从 `~/.hermes/.env` 解析 `${MIOSA_API_KEY}`。请参阅该块的[概述](/docs/mcp#hermes)。

## 从 CLI 安装或提供服务

`miosa` CLI 可以为您配置或运行 MCP 服务器，因此您不必手动编辑客户端配置。

```bash
miosa mcp install                                  # Claude Code, user scope
miosa mcp install --client cursor                  # Cursor
miosa mcp install --client gemini                  # Gemini CLI
miosa mcp install --client claude --scope project  # a shared .mcp.json
miosa mcp install --print                          # show the change, write nothing
```

`miosa mcp install` 不写入 OSA 或 Hermes 配置；使用上面的两个部分。它将托管端点和 API 密钥写入客户端的配置文件，并保留已列出的其他服务器。需要 `--client`（`claude`、`cursor`、`gemini` 或 `manual`）、`--name`（默认 `miosa`）、`--scope`（`user` 或 `project`，克劳德）、`--url`、`--force` 和 `--print`。

对于需要 stdio 服务器的客户端，请通过标准输入运行桥接：

```bash
miosa mcp serve
```

`miosa mcp serve` 读取以换行符分隔的 JSON-RPC 2.0 消息，将每条消息转发到 MIOSA MCP，并为每个响应写入一个 JSON 行。向客户指出：

```json
{
  "mcpServers": {
    "miosa": {"command": "miosa", "args": ["mcp", "serve"]}
  }
}
```

列出服务器公开的工具：

```bash
miosa mcp tools
```

`miosa mcp serve` 和 `miosa mcp tools` 均采用 `--url`（默认值：API 基本 URL 加 `/mcp`）。 Python 包还公开了同一 stdio 角色的 `miosa-mcp` 入口点。

有关每个客户端的配置矩阵、环境变量参考（仅限本地服务器）和故障排除，请参阅[高级 MCP 设置](/docs/mcp/advanced)。

## 获取钥匙

在 [miosa.ai/dashboard/api-keys](https://miosa.ai/dashboard/api-keys) 创建用户 API 密钥。它看起来像`msk_u_<hex>`。将其视为密码。

从 CLI：

```bash
miosa api-key create mcp-key --preset agent
miosa api-key create mcp-key --preset agent --quiet-key   # print only the secret
```

## 组织背景

托管端点将每个请求的范围限定为拥有 API 密钥的组织，因此那里没有组织切换工具。本地 stdio 包（`miosa-mcp`）是注册组织助手并读取初始组织的 `MIOSA_TENANT` 的包：

```text
organization_list()                       # organizations available to you
organization_current()                    # the organization in use
organization_switch(organization="acme")  # switch by UUID or slug
```

这些工具属于本地软件包，而不是托管服务器； [工具参考](/docs/mcp/tools)列出了托管工具。

> **注意**
>
> 密钥由保护每个 REST 端点的同一身份验证插件进行检查，因此 MCP 表面继承租户范围、计划限制、信用检查和审核日志记录。

## 验证

```bash
claude mcp list
```

您应该看到具有连接状态的 `miosa`。如果调用返回 `401`，则密钥无效、过期或已撤销。 `429` 表示您达到了计划的速率限制（与 REST API 相同的限制；窗口是每分钟）。

您还可以通过网络列出目录：

```bash
curl -sS -X POST https://api.miosa.ai/api/v1/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer msk_u_your_key_here" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | jq .
```
