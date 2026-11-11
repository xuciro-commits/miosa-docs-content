---
slug: "quickstart/sandbox-to-deployment"
title: "从沙箱到部署"
description: "完整的上手演练：创建沙箱、编写代码、实时预览、发布到生产环境，并验证部署正在运行。"
section: "Get started"
order: 2
---

请从**路线 1：从沙箱到部署**开始。
这是推荐的首个演练。
你的第一次成功，是让一个小型站点在 MIOSA 返回的生产 URL 上运行，并验证其部署。
在添加智能体编排或其他运行时之前，请先完成这条路径。

> **正在构建白标产品？** 如果你的目标是推出一个客户永远看不到 MIOSA 品牌的 SaaS，请改从[白标集成商指南](/docs/integrations)开始。该指南从零介绍按用户预配、自定义域名、iframe 嵌入、webhook 和用量计费。

## 前提条件

### 账户和工作区

在 [miosa.ai](https://miosa.ai) 创建 MIOSA 账户，并创建一个工作区。

### API 密钥

在目标工作区的 Settings → API Keys 中创建密钥。
密钥类型和作用域请参阅 [API 密钥](/docs/platform/api-keys)。
在运行示例的终端中导出密钥：

```bash
export MIOSA_API_KEY="msk_u_..."
```

### 安装所用语言的工具

**TypeScript**

```bash
npm install @miosa/cli @miosa/sdk
# or: pnpm add @miosa/cli @miosa/sdk
```

**Python**

```bash
pip install --upgrade miosa
```

**Elixir**

```elixir
# mix.exs  -  published on Hex.pm: https://hex.pm/packages/miosa
{:miosa, "~> 1.1"}
```

然后运行 `mix deps.get`。

**MCP（Claude Code / Cursor / Windsurf）**

**零安装选项——只需一个 URL 和 API 密钥：**

```json
{
  "mcpServers": {
    "miosa": {
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": {
        "Authorization": "Bearer msk_u_..."
      }
    }
  }
}
```

将其添加到 `~/.claude/mcp.json`（Claude Code）、Cursor 的 MCP 设置或 Windsurf 的 `mcp_config.json`。无需 pip，无需 npm——客户端重启后，当前按能力划分的工具目录会立即出现。这是最简单的上手方式。

**本地服务器（可选）：**

```bash
pip install git+https://github.com/Miosa-osa/miosa-mcp.git
```

或通过 TypeScript CLI：

```bash
npx @miosa/cli mcp serve
```

> **注意：绝不要把 msk_* 放进浏览器**
>
> 工作区 API 密钥仅限服务端使用。若要在浏览器端嵌入预览，由你的后端签发短期分享令牌。请参阅[浏览器令牌](/docs/platform/browser-tokens)。

## 第 1 步——创建沙箱。

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  workspaceSlug: "default", // the workspace it belongs to; every create names one
})
console.log(sandbox.id, sandbox.state)
// sbx_a1b2c3d4e5  running
```

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sandbox = miosa.sandboxes.create(template_id="miosa-sandbox", workspace_slug="default")
print(sandbox.id, sandbox.state)
# sbx_a1b2c3d4e5  running
```

## 第 2 步——在 `/workspace` 中写入一个小型静态站点。

**TypeScript**

```typescript
await sandbox.files.write(
  "/workspace/index.html",
  `<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>`
)
```

**Python**

```python
sandbox.files.write(
    "/workspace/index.html",
    """<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>""",
)
```

## 第 3 步——启动开发服务器，以便实时预览。

**TypeScript**

```typescript
// Start the dev server. `exec.stream` keeps the process running and yields its output.
const events = sandbox.exec.stream("python3 -m http.server 3000 --bind 0.0.0.0", { cwd: "/workspace" })
for await (const event of events) console.log(event)
```

**Python**

```python
# Start the dev server in the background.
server = sandbox.commands.run(
    "python3 -m http.server 3000 --bind 0.0.0.0",
    cwd="/workspace",
    background=True,
)
```

## 第 4 步——创建预览并打开。

**TypeScript**

```typescript
const preview = await sandbox.previews.create(3000)
console.log(preview.url)
// https://3000-abc12345.sandbox.miosa.ai
```

**Python**

```python
preview = sandbox.previews.create(port=3000)
print(preview.url)
# https://5173-abc12345.sandbox.miosa.ai
```

   在浏览器中打开该 URL。你会看到 `index.html` 从沙箱内部实时提供。

## 第 5 步——发布到生产环境。

**CLI**

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name my-app \
  --dir /workspace \
  --type docker_deploy \
  --wait \
  --json
```

使用 MIOSA 返回的 `deployment.id` 和 `deployment.public_url`。

**TypeScript**

```typescript
const result = await sandbox.deploy({
  name: "my-app",
  outputPath: "/workspace",
})
// Read the production URL from the response; do not construct it.
console.log(result)
```

**Python**

```python
result = sandbox.deploy(
    name="my-app",
    output_path="/workspace",
)
# Read the production URL from the response; do not construct it.
print(result)
```

   MIOSA 会冻结源快照，运行构建器打包文件，存储一个不可变的发布版本，并将部署 URL 路由到该版本。沙箱保持存活且可编辑。

## 第 6 步——验证部署。
生产 URL 是 MIOSA 返回的 URL。
不要用 slug 自行拼接。
当你需要让操作员或 CI 检查应用是否在线时，请使用部署证明。

```bash
miosa deploy get <deployment-id> --json
```

对于 Docker Host 部署，证明会检查部署记录、Docker Host 主机链接、主机健康状态、持久应用目标、容器路由和公共 URL 探测。

## 第 7 步——修改并重新发布。
再次编辑 `/workspace/index.html`，并发布到现有部署：

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name <deployment> \
  --dir /workspace \
  --type docker_deploy \
  --wait \
  --json
```

MIOSA 会创建一个新的不可变版本，并且只在验证成功后将其提升为生产版本。
之前健康的版本仍可用于回滚。

## 第 8 步——（可选）绑定自定义域名。
   DNS 设置请参阅[域名](/docs/docker-host/domains)。简而言之：从你的域名添加一条指向生成主机的 `CNAME`，调用验证端点，MIOSA 会签发 TLS 证书。

## 第 9 步——清理。

**TypeScript**

```typescript
await sandbox.destroy()
```

**Python**

```python
sandbox.destroy()
```

   沙箱已删除。已发布的部署及其域名不受影响——发布版本不可变，独立于沙箱存放在对象存储中。

### 你刚刚完成了什么

| 步骤 | 发生了什么 |
|------|--------------|
| `sandboxes.create` | 在沙箱镜像上创建隔离的沙箱 |
| `files.write` | 将文件写入运行中 VM 内的 `/workspace` |
| `exec` | 通过 exec API 在 VM 内运行命令 |
| `previews.create` | 打开从公共主机名到 VM 内 3000 端口的隧道 |
| `sandbox publish` | 冻结源、运行构建器、存储发布版本、创建版本并使其上线 |
| `deploy prove` | 验证 MIOSA 能够证明部署路由和运行时状态 |
| `sandbox.destroy` | 终止 VM；部署不受影响 |

## 接下来做什么

- **添加真实后端。** 通过[发布](/docs/docker-host/publishing)和托管 [Postgres 数据库](/docs/data/postgres)从静态部署转向动态部署。
- **验证最终用户身份。** [身份验证即服务](/docs/data/auth)只需一个环境变量，即可加入完整的 JWT 注册/登录流程。
- **回滚有问题的部署。** [回滚](/docs/docker-host/rollback)可在数秒内将域名重新指向较早版本。
- **理解每个基础概念。** [概念](/docs/concepts)是标准的架构参考。
- **在 MIOSA 上构建你自己的产品。** [平台 / 白标概览](/docs/platform/overview)涵盖预览令牌、分享 URL、租户品牌、按用户配额、webhook 订阅、Docker Host 域名和审计日志。
- **接入 UI 组件。** `@miosa/react`、`@miosa/vue` 和 `@miosa/svelte` 提供 `MiosaPreview`、`MiosaTerminal`、`MiosaFileTree` 和 `MiosaUsage`。请参阅 [UI 组件](/docs/ui-components)。
- **查看完整的白标示例。** [参考集成](https://github.com/Miosa-osa/miosa/tree/main/examples)展示了一个完整的平台嵌入方案。
- **用 MIOSA 技能扩展 Claude Code。** [github.com/Miosa-osa/miosa-skills](https://github.com/Miosa-osa/miosa-skills) 提供 5 个预构建技能，为 Claude Code 带来沙箱、计算机和部署基础能力。请参阅[技能](/docs/guides/skills)。
- **API 传输格式。** [API 参考](/docs/api-reference)记录了每个端点。
