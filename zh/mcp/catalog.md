---
slug: "mcp/catalog"
title: "工具服务器目录"
description: "一步添加官方工具服务器，保存前测试其密钥，并将密钥保存在工作区保管库中。"
section: "CLI, SDKs & MCP"
order: 11031
---

MIOSA 保留了一份官方工具服务器目录，您可以一步添加，然后附加到您的智能体。
从 **连接** 打开它，然后 **工具服务器**，然后 **浏览目录**。

## 目录中有什么

该目录列出了 40 个经过验证的工具服务器，包括 Apify、GitHub、Linear、Notion、Sentry、Stripe、Cloudflare、Firecrawl、Exa、Perplexity、Context7 和 Playwright。

按名称搜索或按类别过滤。
每个条目都显示供应商需要什么以及从哪里获得。

某些服务器仅通过浏览器登录，例如 Figma、Vercel、Google Workspace 和 Mobbin。
它们已列出，但没有关键字段，并且尚不可用。

## 添加一个密钥并测试它

添加服务器需要提供供应商所需的密钥，每个密钥都有一个“在此处获取”链接。
在保存任何内容之前，**测试连接**会根据供应商检查密钥。
密钥仅发送到供应商自己的 URL，并且测试不存储任何内容。

成功后，密钥将密封在工作区保管库中，并且保存的服务器保存 `{{secret:NAME}}` 引用，而不是值。
响应永远不会携带密钥。

如果该名称已被占用，则会出现明显的冲突，您可以使用自己的名称添加第二个帐户。

## 今天哪些利用加载工具服务器

工具服务器由运行框架加载，其 CLI 读取工具服务器配置。
今天，这就是 Claude Code、Codex 和 OSA。
通用工具 pi、OpenCode、Prime Agent、Kimi Code 和 Mistral Vibe 尚未加载工具服务器。

## 作为 API 的目录

|路线 |范围 |它有什么作用 |
|---|---|---|
| `GET /api/v1/mcp-catalog?workspace_id=` | `connections:read` |列出条目、类别以及您已添加的内容 |
| `POST /api/v1/mcp-catalog/:slug/test` | `connections:write` |根据供应商的 URL 测试密钥，并且不存储任何内容 |
| `POST /api/v1/mcp-catalog/:slug/install` | `connections:write` |添加服务器并将其密钥密封在工作区保管库中 |

Install 发送 `{ workspace_id, method, name, values }` 并返回 `201` 以及服务器和机密名称，而不是值。
重复返回 `409 ALREADY_ADDED`。
错误字段返回 `422 VALIDATION_ERROR`，其中包含每个字段的详细信息。

## 相关

- [MCP概述](/docs/mcp)：您自己的智能体调用的MIOSA工具服务器。
- [Harnesses and auth](/docs/agents/harnesses)：哪些智能体在您的帐户上运行。
- [托管连接器](/docs/guides/managed-connectors)：MIOSA 拥有的工具，例如 Refero。
