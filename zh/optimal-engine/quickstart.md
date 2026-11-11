---
slug: "optimal-engine/quickstart"
title: "Optimal Engine快速入门"
description: "在您自己的计算机上运行 Optimal Engine、创建 API 密钥、创建工作区、存储源并进行搜索（通过结帐或使用 Docker）以及 MIOSA 上的最短路径。"
section: "Optimal Engine"
order: 8002
---

本指南在您的计算机上运行一个引擎、创建工作区、存储源并再次查找它。
大约需要十分钟，大部分是第一次编译。
要在 MIOSA 上使用托管引擎，请跳至 [在 MIOSA](/docs/optimal-engine/quickstart#on-miosa)。

你以：

- `http://127.0.0.1:4200` 上的引擎；
- 涵盖每个请求的 API 密钥；
- 一个工作空间 `default:notes` 与一个存储源；
- 指向该来源的搜索结果。

## 1. 运行引擎

**从结帐处**

您需要 Erlang/OTP 27 或更高版本、Elixir 1.17 或更高版本、git 和 C 工具链。
在 macOS 上，还运行 `brew install snappy`。

```bash
git clone https://github.com/Miosa-osa/OptimalEngine.git
cd OptimalEngine
mix deps.get
```

在服务器启动之前创建第一个密钥。
密钥已打印一次，因此请立即复制。

```bash
mix optimal.auth mint --name "root" --workspace '*'
```

使用 HTTP API 和所需密钥启动引擎：

```bash
OPTIMAL_API_ENABLED=true OPTIMAL_AUTH_REQUIRED=true mix run --no-halt
```

第一次启动会在 API 响应之前运行每个数据库迁移。

**使用 Docker**

你需要 Docker。
第一个映像构建编译 RocksDB，可能需要 30 到 60 分钟。

```bash
git clone https://github.com/Miosa-osa/OptimalEngine.git
cd OptimalEngine
docker build -f deploy/Dockerfile.engine \
  --build-arg OPTIMAL_RELEASE_CHANNEL=stable \
  -t optimal-engine:quickstart .
```

在一次性容器中创建第一个密钥，该容器在 API 关闭的情况下打开相同的卷：

```bash
docker run --rm -v optimal-data:/data -e OPTIMAL_API_ENABLED=false optimal-engine:quickstart \
  /app/bin/optimal eval \
  'OptimalEngine.Auth.ApiKey.mint(%{tenant_id: "default", name: "root", scopes: ["*"], workspace_scope: ["*"]}) |> IO.inspect()'
```

启动发动机。
`OPTIMAL_ENGINE_API_KEY` 是您刚刚创建的密钥；容器健康检查使用它。

```bash
docker run -d --name optimal-engine -p 127.0.0.1:4200:4200 \
  -v optimal-data:/data \
  -e OPTIMAL_AUTHORITY_KEY="$(openssl rand -hex 32)" \
  -e CONNECTOR_KEY="$(openssl rand -hex 32)" \
  -e OPTIMAL_ENGINE_API_KEY="<your key>" \
  optimal-engine:quickstart
```

该镜像开启API，默认需要密钥。
引擎存储的所有内容都位于 `optimal-data` 卷中，因此更换容器会保留您的数据。

## 2. 检查它是否有答案

在第二个终端中：

```bash
export OPTIMAL_ENGINE_API_KEY="<your key>"
curl -s http://127.0.0.1:4200/api/health -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
```

```json
{
  "status": "up",
  "live": true,
  "checks": {
    "store": ":ok",
    "migrations": ":ok",
    "credential_key": ":ok",
    "network_auth": ":ok"
  },
  "degraded": [],
  "warnings": []
}
```

当出现警告时，`status` 为 `up`、`degraded`，当检查失败时为 `down`。
真实的响应还带有发布标识。

## 3. 创建工作区

```bash
curl -s -X POST http://127.0.0.1:4200/api/workspaces \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"slug": "notes", "name": "Notes"}'
```

响应是新的工作空间。
它的`id`是`default:notes`；在下面的所有地方使用该 id。

## 4. 存储源

```bash
curl -s -X POST http://127.0.0.1:4200/api/ingest \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace": "default:notes",
    "title": "Supplier renewal call",
    "text": "Renewal call with the supplier moved to the 14th. The contract review is still open."
  }'
```

```json
{
  "ok": true,
  "signal_id": "...",
  "genre": "...",
  "type": "...",
  "entities": [],
  "source_package_id": "..."
}
```

引擎保留了来源，对其进行了分类，并提出了索赔要求。
一旦经过审核或符合工作区的升级规则，声明就会成为事实；请参阅[概念](/docs/optimal-engine/concepts#claims-and-facts)。

## 5. 搜索并询问

```bash
curl -s "http://127.0.0.1:4200/api/search?q=supplier%20renewal&workspace=default:notes&limit=5" \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
```

每个结果都有一个 `title`、匹配的 `content`、`score` 和 `optimal://` 方案中的 `uri` 指向源。

```bash
curl -s -X POST http://127.0.0.1:4200/api/rag \
  -H "Authorization: Bearer $OPTIMAL_ENGINE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "What is open with the supplier?", "workspace": "default:notes", "format": "markdown"}'
```

`envelope.body` 是应写入答案的上下文，`envelope.sources` 列出了它使用的来源。
引擎本身不会写出答案；将信封交给您的模特或经纪人。

> **提示：矢量搜索是可选的**
>
> 开箱即用的全文搜索。
> 使用 `nomic-embed-text` 运行 [Ollama](https://ollama.com/)，或将 `OPTIMAL_EMBED_URL` 指向嵌入端点，以添加矢量搜索。
> 请参阅[安装和配置](/docs/optimal-engine/install#embeddings)。

## 6. 使用客户端 CLI

该存储库提供了一个小型客户端 CLI，`optimal`（也称为 `oe`），仅需要 `sh`、`curl` 和 `awk`。

```bash
apps/cli/install.sh
export OPTIMAL_ENGINE_URL=http://127.0.0.1:4200
optimal doctor
optimal search "supplier renewal" -w default:notes
```

有关每个命令以及通过 MCP 连接智能体的信息，请参阅 [CLI、SDK 和 MCP](/docs/optimal-engine/cli-sdks-mcp)。

## 关于 MIOSA

每个组织都启用托管引擎。
一旦您的启用：

1. 在控制台中打开 **Optimal > Memory** 并选择一个工作区。
2. 按**启动内存**。控制台在创建任何内容之前会显示大小和每小时价格。
3.当Memory运行时，在同一页面搜索它，或者在聊天中要求Optimal将笔记保存到Memory。

从终端，[MIOSA CLI](/docs/cli) 执行相同的操作：

```bash
miosa engine create notes-brain --workspace notes --for workspace --wait
miosa engine connect notes-brain
optimal search "supplier renewal"
```

请参阅 [MIOSA 上的Optimal Engine](/docs/optimal-engine/miosa) 了解其中包含的内容以及如何请求访问。
