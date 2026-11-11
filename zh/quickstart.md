---
slug: "quickstart"
title: "快速开始"
description: "安装 CLI、启动沙箱、让智能体在其中构建应用，并通过实时 URL 打开它。"
section: "Get started"
order: 1
---

MIOSA 为您的代码和 AI 智能体运行隔离的 Linux 沙箱和完整云桌面，然后将结果发布到实时 URL。
本页用四个步骤带您从安装走到由智能体构建的应用。

视频：[快速开始讲解](https://miosa.ai/videos/quickstart-explained.mp4)（1:35）

## 让您的智能体驱动 MIOSA

大多数人通过编码智能体来使用 MIOSA，而不是手动操作。
您的智能体可以运行在笔记本电脑上，并控制整个平台。
它有四种接入方式：

| 接入方式 | 操作 |
|---|---|
| CLI | 您的智能体运行 `miosa` 命令。请从 [Agent 操作指南](/docs/guides/agent-operating-guide) 开始。 |
| MCP server | `miosa mcp install --client claude`（也支持 `cursor` 和 `gemini`）。OSA 和 Hermes 改用配置文件：请参阅 [MCP](/docs/mcp)。 |
| Skills | 适用于沙箱、计算机和部署的 Claude Code 技能。请参阅 [Claude Code 技能](/docs/guides/skills)。 |
| `llms.txt` | 让任何智能体指向 [miosa.ai/llms.txt](https://miosa.ai/llms.txt)。它以纯文本列出每个文档页面。 |

这适用于 OSA、Claude Code、Codex、OpenCode、Hermes、Kimi Code 等。
本页的其余部分会把智能体放进沙箱。

已经知道自己需要什么？

- [从沙箱到部署](/docs/quickstart/sandbox-to-deployment)：推荐的第一个完整演练。
- [智能体驱动的构建](/docs/quickstart/agent-driven-build)：从您的后端分发工作、流式接收事件并收集产物。
- [Docker Host](/docs/quickstart/app-engine)：在工作区 Docker Host 主机上运行容器化应用；请先查看[可用性](/docs/availability)。
- [连接您自己的服务器](/docs/opencomputers/connect)：在您运营的 Linux 机器上运行第一条命令。

## 1. 安装

**macOS / Linux**

```bash
curl -fsSL https://raw.githubusercontent.com/Miosa-osa/miosa-cli-go/main/install.sh | sh
```

**Windows**

```powershell
npm i -g @miosa/cli
```

安装程序会执行以下步骤：

1. 检测您的操作系统和 CPU。
2. 下载匹配的 `miosa` 二进制文件并校验其 SHA-256。
3. 将其安装到 `~/.local/bin/miosa`（以 root 身份运行时安装到 `/usr/local/bin`）。

然后运行 `miosa login` 登录：它会打开浏览器，这台机器的密钥会保存在 `~/.miosa/config.toml`。
更想用 npm、Homebrew 或手动下载？请参阅[安装 CLI 或 SDK](/docs/cli/install)。

## 2. 启动沙箱

```bash
miosa create my-box --wait     # provision a sandbox and wait until it is ready
miosa list                     # see all your sandboxes
```

沙箱是一台快速、隔离的 Linux VM。
您的智能体向其中写入文件、运行命令，并将端口开放为实时预览 URL。
沙箱暂停时其文件系统会保留，因此您可以稍后恢复它。
请阅读[沙箱](/docs/develop/sandboxes)了解规格、限制和生命周期。

## 3. 在其中运行智能体

选择您的智能体。
MIOSA 会在沙箱中安装它并在那里运行。
智能体使用您自己的模型账户，因此您向自己的提供商付费。
MIOSA 积分从不会用于支付模型费用。

每个智能体都包含三个部分：连接您的账户、启动智能体，以及给它一个任务。

**OSA**

OSA 是 MIOSA 自己的智能体。它使用您连接的模型。

**连接。**
打开 MIOSA 控制台，进入 **Agents**，为 OSA 连接您的模型密钥。

**启动并给它一个任务。**

```bash
miosa prompt --harness osa --sandbox my-box \
  "Build a landing page and serve it on port 3000"
```

**Claude Code**

**连接。**
使用您的 Claude 套餐登录。
命令会打印一个链接和一个验证码。在浏览器中批准，然后粘贴回验证码。

```bash
miosa connections add models claude --sandbox my-box
```

或使用您自己的 Anthropic 密钥：

```bash
miosa connections add models anthropic --key-stdin <<< "$ANTHROPIC_API_KEY"
```

**启动并给它一个任务。**

```bash
miosa prompt --harness claude-code --sandbox my-box \
  "Build a landing page and serve it on port 3000"
```

**Codex**

**连接。**
使用您的 ChatGPT 套餐登录。
命令会打印一个链接和一个验证码。在浏览器中批准。

```bash
miosa connections add models chatgpt --sandbox my-box
```

或使用您自己的 OpenAI 密钥：

```bash
miosa connections add models openai --key-stdin <<< "$OPENAI_API_KEY"
```

**启动并给它一个任务。**

```bash
miosa prompt --harness codex --sandbox my-box \
  "Build a landing page and serve it on port 3000"
```

**OpenCode**

**连接。**
在 MIOSA 控制台中进入 **Agents**，为 OpenCode 连接以下之一：Anthropic、OpenAI、OpenRouter 或 Mistral 密钥，或您的 ChatGPT 套餐。
`miosa connections` 尚不支持此智能体。

**启动并给它一个任务。**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"opencode\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

**Hermes**

Hermes Agent 是运行 API 中的一等运行器。
它尚未出现在 CLI 的 `--harness` 列表中，因此请使用运行 API 启动它。

**连接。**
在 MIOSA 控制台中进入 **Agents**，连接一个 ChatGPT 套餐、一个 Claude 套餐（带额外用量的 Max），或 Anthropic、OpenAI、OpenRouter 密钥。
`miosa connections` 尚不支持此智能体。

**启动并给它一个任务。**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"hermes\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

**Kimi Code**

**连接。**
在 MIOSA 控制台中进入 **Agents**，连接您的 Kimi Code 套餐或 Moonshot 密钥。
`miosa connections` 尚不支持此智能体。

**启动并给它一个任务。**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"kimi\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

**Pi**

**连接。**
在 MIOSA 控制台中进入 **Agents**，为 Pi 连接以下之一：Anthropic、OpenAI、OpenRouter 或 Mistral 密钥，或您的 Claude 或 ChatGPT 套餐。
`miosa connections` 尚不支持此智能体。

**启动并给它一个任务。**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"pi\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

**Mistral**

**连接。**
在 MIOSA 控制台中进入 **Agents**，连接您的 Mistral Vibe 套餐或 Mistral 密钥。
`miosa connections` 尚不支持此智能体。

**启动并给它一个任务。**

```bash
SANDBOX_ID=$(miosa info my-box --json | jq -r .id)
RUN=$(miosa api POST /runs -d "{
  \"runner\": \"mistral\",
  \"sandbox_id\": \"$SANDBOX_ID\",
  \"instruction\": \"Build a landing page and serve it on port 3000\"
}" | jq -r .data.id)
miosa run follow $RUN
```

如果您尚未连接账户，运行会被拒绝或失败。
MIOSA 绝不会回退到自己的密钥。

## 4. 查看它构建了什么

```bash
miosa preview create my-box 3000 --visibility public
```

打开它打印的 URL。
那就是您的智能体构建的页面，从沙箱内部提供服务。

如果没有任何内容加载，说明智能体没有让服务器保持运行。
您可以自己启动一个：

```bash
miosa process start my-box --name web --cwd /workspace -- python3 -m http.server 3000 --bind 0.0.0.0
```

## 或者手动操作

您不需要智能体也能完成这些。
自己写一个文件、启动一个服务器，然后打开预览。

**CLI**

```bash
miosa exec my-box -- echo "hello from MIOSA"
echo "<h1>Hello</h1>" | miosa files write my-box:/workspace/index.html
miosa process start my-box --name web --cwd /workspace -- python3 -m http.server 3000 --bind 0.0.0.0
miosa preview create my-box 3000 --visibility public
```

`miosa process start` 会立即返回，并让服务器继续运行。
不要用 `miosa exec` 来启动服务器，因为 `exec` 会等待命令结束。

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  workspaceSlug: "default", // the workspace it belongs to; every create names one
  wait: true, // hold the call until the sandbox is ready
})
console.log(sandbox.id, sandbox.state)
// sbx_a1b2c3d4e5  running

await sandbox.files.write(
  "/workspace/index.html",
  `<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>`
)

// Start the server in the background. This call returns right away.
await sandbox.exec(
  "nohup python3 -m http.server 3000 --bind 0.0.0.0 > /tmp/web.log 2>&1 &",
  { cwd: "/workspace" }
)

const preview = await sandbox.previews.create(3000)
console.log(preview.url)
// https://3000-abc12345.sandbox.miosa.ai
```

您也可以把整个任务交给智能体：`await sandbox.run("Build a landing page and serve it on port 3000", { runner: "codex" })`。

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sandbox = miosa.sandboxes.create(
    template_id="miosa-sandbox",
    workspace_slug="default",  # the workspace it belongs to; every create names one
    wait=True,
)
print(sandbox.id, sandbox.state)
# sbx_a1b2c3d4e5  running

sandbox.files.write(
    "/workspace/index.html",
    """<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>""",
)

# Start the server in the background. This call returns right away.
sandbox.processes.start("python3 -m http.server 3000 --bind 0.0.0.0", name="web")

preview = sandbox.previews.create(port=3000)
print(preview["url"])
# https://3000-abc12345.sandbox.miosa.ai
```

打开 MIOSA 返回的预览 URL。您的 `index.html` 会从沙箱内部实时提供服务。

## 按需选择

| 您想要 | 产品 | 从这里开始 |
|---|---|---|
| 运行代码、构建、测试和开发服务器 | [沙箱](/docs/develop/sandboxes) | [创建并连接](/docs/develop/sandboxes/create) |
| 驱动浏览器、桌面应用、截图 | [计算机](/docs/computers/overview) | [计算机概览](/docs/computers/overview) |
| 通过公共 URL 发布常驻应用 | [Docker Host](/docs/docker-host/docker-deploy) | [部署概览](/docs/docker-host/overview) |
| 针对沙箱或计算机运行 AI 智能体 | [智能体运行](/docs/api-reference/agent-runs) | [Agent 操作指南](/docs/guides/agent-operating-guide) |
| 在 MIOSA 上构建白标产品 | [平台构建者](/docs/platform/overview) | [平台概览](/docs/platform/overview) |

## 后续

- [从沙箱到部署](/docs/quickstart/sandbox-to-deployment)：完整的首次演练：创建沙箱、写入文件、预览、发布到生产环境，并验证部署确实在线。
- [智能体驱动的构建](/docs/quickstart/agent-driven-build)：从您的后端向智能体分发工作、流式接收事件并收集产物，同时带上外部归因。
- [Docker Host](/docs/quickstart/app-engine)：在工作区 Docker Host 主机上运行容器化应用。
- [连接您自己的服务器](/docs/opencomputers/connect)：在您运营的 Linux 机器上运行第一条命令。

- [概念](/docs/concepts)：一份架构参考，涵盖 MIOSA 的每个基础构件。
- [API 参考](/docs/api-reference)：每个端点、错误代码和 Webhook。
