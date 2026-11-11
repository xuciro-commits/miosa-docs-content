---
slug: "guides/managed-connectors"
title: "沙盒智能体的托管连接器"
description: "将 MIOSA 管理的工具（例如 Refero）附加到沙箱或计算机智能体，无需客户端提供的供应商令牌。"
section: "Sandboxes"
order: 2020
---

MIOSA 智能体应构建在 MIOSA 沙箱或计算机内。本地 CLI 或
MCP 客户端应协调工作；沙箱/计算机应该位于
智能体写入文件、安装包、运行命令、预览应用程序以及
产生文物。

托管连接器为这些虚拟机内智能体提供工具，而无需要求每个客户端
带上供应商凭证。

## 连接器、出口和 MCP

这三个表面一起工作，但它们不可互换：

|表面|拥有 |示例|
|---|---|---|
|连接器|为提供商或工具授予身份和凭证 | `refero/design-research`、`anthropic/workspace-claude`，租户 OAuth 安装 |
|出口 |从沙箱/计算机到批准端点的网络策略和令牌智能体 |仅在调用 `https://api.refero.design/mcp` | 时交换 `miosa-tok-*`
| MCP|智能体运行时在沙箱、计算机、编辑器或托管端点内使用的工具协议 | Claude Code 调用 Refero MCP 服务器或 MIOSA 托管的 MCP 服务器 |

连接器显示**授予什么帐户/工具**。出口表示**所在网络
流量可能会消失以及何时可以交换占位符**。 MCP 说**如何
智能体发现并调用工具**。产品可以一次使用所有三个：
连接连接器，强制出口到提供者端点，并配置
智能体运行时通过 MCP 调用该提供程序。

> **提示：内置提供程序**
>
> `refero/design-research` 由 MIOSA 管理。租户可以将其附加到沙箱，
> 但 MIOSA 不会将 Refero 令牌返回给客户端。

## 默认流程

创建一个有足够时间用于主动智能体工作的沙箱：

```bash
miosa create builder \
  --template nextjs \
  --ttl 1h \
  --wait \
  --json
```

附上 Refero 设计研究：

```bash
miosa connections add tools \
  --name refero-design-research \
  --url https://api.refero.design/mcp \
  --header-env Authorization=REFERO_MCP_TOKEN \
  --json
```

在 `/workspace` 内运行智能体：

```bash
miosa prompt --sandbox <sandbox-id> \
  --harness claude \
  --json \
  "Research product references, build the page, run tests, and leave files in /workspace"
```

对于直接使用工具，可以使用文件和命令编辑同一沙箱
命令：

```bash
miosa files cp ./page.jsx <sandbox-id>:/workspace/app/page.jsx --json
miosa exec <sandbox-id> --cwd /workspace --json -- npm run build
miosa wait <sandbox-id> --timeout 180s --json
```

## 代币如何运作

沙箱接收一个智能体占位符：

```bash
REFERO_MCP_TOKEN          # 由 MIOSA 注入，请勿填写
```

真正的提供商令牌在 MIOSA 控制的基础设施中保持加密状态。的
出口智能体仅交换其所属的提供者端点的占位符，
对于推荐人：

```text
https://api.refero.design/mcp
```

这意味着：

- 客户不粘贴 Refero 令牌，
- `miosa connections list` 未显示令牌，
- `miosa connections test refero-design-research` 不返回令牌，
- 沙箱代码仍然可以使用基于正常环境变量的 MCP 配置。

## 沙箱内的 MCP 配置

在智能体的 MCP 配置中使用智能体的环境变量：

```toml
[mcp_servers.refero]
enabled = true
url = "https://api.refero.design/mcp"

[mcp_servers.refero.http_headers]
Authorization = "Bearer ${REFERO_MCP_TOKEN}"
```

## 租户提供的连接器

当 MIOSA 不提供连接器时，租户仍然可以自带连接器：

```bash
printf '%s' "$ANTHROPIC_API_KEY" \
  | miosa connections add models anthropic \
      --label workspace-claude \
      --key-stdin \
      --json
```

## 我应该使用哪个运行时？

- [沙盒](/docs/develop/sandboxes)：智能体编码、生成文件、测试、包安装、实时预览和发布准备。
- [计算机](/docs/computers/overview)：持久桌面或虚拟机工作流程、浏览器/GUI 任务、SSH 和长时间运行的操作员会话。
- [Docker Host](/docs/docker-host/docker-deploy)：沙箱构建的应用程序准备就绪后，工作区 Docker Host 主机上的持久应用程序容器。
