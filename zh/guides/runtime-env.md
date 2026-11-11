---
slug: "guides/runtime-env"
title: "智能体设备的运行时环境"
description: "为租户、工作区或项目范围内的沙箱、计算机和智能体运行时定义继承的环境变量。"
section: "Agents"
order: 6013
---

Runtime env 让平台所有者定义一次环境变量并拥有
它们由沙箱、计算机和智能体运行继承。使用它作为提供商密钥，
模型配置、MCP 代币、工作区默认值和白标客户
当设备启动或运行时应自动可用的设置
命令。

这是用户创建许多智能体工作区的产品的基础：
应用程序构建者、浏览器运营商、24/7 公司智能体、研究智能体和
客户特定的自动化车队。

> **注意：回复已被编辑**
>
> 运行时环境值在静态时被加密，并且永远不会以明文形式返回
> API、CLI 或 SDK。响应仅包括元数据和简短预览。

## 范围模型

MIOSA 按以下顺序解析运行时环境：

|优先级 |范围 |适用于 |
|---:|---|---|
| 1 |租户|租户下的每个工作区、项目、沙箱、计算机和智能体 |
| 2 |工作空间 |该工作区中的每个项目、沙箱、计算机和智能体 |
| 3 |项目|归因于该项目的沙箱、计算机、部署和智能体运行 |
| 4 |资源 |直接在资源上设置每沙箱或每计算机环境 |
| 5 |运行| Env 直接传递给单个 exec 或智能体运行 |

更具体的值会覆盖具有相同名称的更广泛的值。项目价值
覆盖工作空间值；工作空间值会覆盖租户值。

## 目标模型

每个运行时环境变量也有一个目标：

|目标|当 | 时使用它
|---|---|
| `all` |该值对于每个运行时表面都是安全且有用的 |
| `sandbox` |该值应该只出现在沙箱 exec/build/agent 会话中 |
| `computer` |该值应该只出现在桌面计算机命令和智能体中 |
| `agent` |该值应注入到在沙箱或计算机上运行的提示驱动智能体中 |

在相同范围内，特定于目标的值会覆盖 `all`。

## 常见模式

- **白标默认**：设置 `ANTHROPIC_API_KEY`、`OPENAI_API_KEY` 或产品功能标志
  一个客户端工作区的工作区范围。
- **项目提供者密钥**：在项目范围内为一个应用程序或客户工作流程设置 `HIGGSFIELD_API_KEY` 或图像模型密钥。
- **智能体运行时配置**：为一类智能体设置 `ANTHROPIC_MODEL`、`MIOSA_AGENT_CWD` 或 MCP 端点配置。
- **沙盒构建秘密**：为生成的构建设置部署、注册表、包管理器或私有 API 凭据。

## 命令行界面

Runtime Env 没有专用的 `miosa` 子命令；用 `miosa api` 驱动它
针对 `/runtime-env` 端点。设置租户范围的沙箱值：

```bash
miosa api POST /runtime-env -d '{
  "name": "ANTHROPIC_API_KEY",
  "value": "sk-ant-...",
  "scope": "tenant",
  "target": "sandbox"
}'
```

设置工作区范围的计算机值：

```bash
miosa api POST /runtime-env -d '{
  "name": "OPENAI_API_KEY",
  "value": "sk-...",
  "scope": "workspace",
  "workspace_id": "ws_123",
  "target": "computer"
}'
```

为所有运行时表面设置项目值：

```bash
miosa api POST /runtime-env -d '{
  "name": "HIGGSFIELD_API_KEY",
  "value": "higgs_...",
  "scope": "project",
  "project_id": "prj_123",
  "target": "all"
}'
```

列表、显示和删除：

```bash
miosa api GET /runtime-env?scope=workspace&workspace_id=ws_123
miosa api GET /runtime-env/<env-id>
miosa api DELETE /runtime-env/<env-id>
```

按目标过滤以查看一个表面的变量：

```bash
miosa api GET /runtime-env?target=sandbox
```

使用继承的 `target=agent` env 运行智能体并下载它创建的文件：

```bash
miosa api POST /runtime-env -d '{
  "name": "ANTHROPIC_API_KEY",
  "value": "sk-ant-...",
  "scope": "project",
  "project_id": "prj_123",
  "target": "agent"
}'

miosa agent run my-agent \
  --sandbox sbx_123 \
  --no-follow

miosa run files <run-id> --json
miosa run download <run-id> <file-id> --output ./report.html
```

## SDK

**打字稿**

```ts
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

await miosa.runtimeEnv.set({
  scope: "workspace",
  workspaceId: "ws_123",
  target: "sandbox",
  name: "ANTHROPIC_API_KEY",
  value: process.env.ANTHROPIC_API_KEY!,
  metadata: { provider: "anthropic" },
});

const vars = await miosa.runtimeEnv.list({
  scope: "workspace",
  workspaceId: "ws_123",
  target: "sandbox",
});
```

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

client.runtime_env.set(
    scope="workspace",
    workspace_id="ws_123",
    target="sandbox",
    name="ANTHROPIC_API_KEY",
    value=os.environ["ANTHROPIC_API_KEY"],
    metadata={"provider": "anthropic"},
)

vars = client.runtime_env.list(
    scope="workspace",
    workspace_id="ws_123",
    target="sandbox",
)
```

## 在沙箱内验证

创建一个沙箱，等待VM会话，然后读取继承的值：

```bash
miosa create env-check \
  --template miosa-sandbox \
  --ttl 1h \
  --wait \
  --json

miosa exec env-check \
  --json \
  -- printf "$ANTHROPIC_API_KEY"
```

仅当您还希望应用程序预览就绪时才使用 `--port`：

```bash
miosa wait <sandbox-id> --timeout 180s --json
```

## 在计算机内部验证

当智能体需要桌面、浏览器会话、GUI 自动化时使用计算机，
或长时间运行状态：

```bash
miosa computer create browser-agent --json
miosa device exec <computer-id> --command 'printf "$OPENAI_API_KEY"' --json
```

## 验证智能体运行内部

智能体运行是调度到沙箱或计算机的提示驱动任务。他们
继承 `target=agent` env，然后应用任何每次运行的 `env` 覆盖。奔跑
记录存储命令元数据、状态、输出、事件和声明的工件
路径。

**打字稿**

```ts
const run = await miosa.agentRuns.run({
  sandboxId: "sbx_123",
  provider: "claude-code",
  cwd: "/workspace",
  prompt: "Create /workspace/report.html with the final proposal.",
  metadata: {
    artifact_paths: ["/workspace/report.html"],
  },
});

const artifacts = await miosa.agentRuns.artifacts(run.id);
const html = await miosa.agentRuns.downloadArtifact(run.id, artifacts[0].id);
```

**Python**

```python
run = client.agent_runs.run(
    sandbox_id="sbx_123",
    provider="claude-code",
    cwd="/workspace",
    prompt="Create /workspace/report.html with the final proposal.",
    metadata={"artifact_paths": ["/workspace/report.html"]},
)

artifacts = client.agent_runs.artifacts(run["id"])
html = client.agent_runs.download_artifact(run["id"], artifacts[0]["id"])
```

## 智能体运行时配置文件和连接器

运行时环境是智能体配置堆栈的一层：

|层|目的|
|---|---|
|运行时环境 |按范围和目标继承的环境变量和提供程序密钥 |
|智能体运行时配置文件 |运行时选择、非秘密默认值、工具、策略、连接器 |
|连接器|经纪提供商身份或管理帐户 |
|出口 |网络政策和智能体代币交换 |
|智能体运行|发送到沙箱或计算机的一个提示/任务 |当运行时需要正常的环境变量时，将提供程序密钥存储在运行时环境中。
当 MIOSA 应智能体提供者授权或托管工具时，请使用连接器。

## 导出生成的工作

当智能体创建文件时，下载声明的智能体运行工件或导出
原始沙箱路径而不是抓取终端输出：

```bash
miosa run files <run-id> --json
miosa run download <run-id> <file-id> --output ./report.pdf
```

```bash
miosa files cp <sandbox-id>:/workspace/out/report.pdf ./report.pdf --json

miosa files cp -r <sandbox-id>:/workspace/out ./customer-deliverables --json
```

对于新的实时 Docker Host 应用程序，公开开发端口并使用显式生产身份进行发布：

```bash
miosa wait <sandbox-id> --json
miosa deploy create --from-sandbox <sandbox-id> \
  --name my-app \
  --dir /workspace \
  --port 3000 \
  --type docker_deploy \
  --wait \
  --json
```

对于以后的更新，请发布到现有的部署 ID，以便稳定的 URL 和发布历史记录保持附加到同一应用程序：

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name <deployment> \
  --dir /workspace \
  --port 3000 \
  --type docker_deploy \
  --wait \
  --json
```
