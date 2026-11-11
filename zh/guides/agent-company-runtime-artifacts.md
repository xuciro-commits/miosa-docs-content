---
slug: "guides/agent-company-runtime-artifacts"
title: "运行时配置文件和工件"
description: "从运行时配置文件到工件导出的端到端通道、提示到设备流程、环境和机密如何继承以及智能体输出如何变成下载。"
section: "Agents"
order: 6022
---

本页介绍智能体产品的运行时机制：运行如何从运行时配置文件开始、提示如何到达设备、环境和机密如何继承，以及生成的文件如何成为面向用户的下载。

## E2E 运行时配置文件到工件导出

这是首先为 Polsia、CoFounder、Nebula、Heuresis、BusinessOS、Lovable、Replit、Genspark、Emergent 或 Blackbox AI 型产品实施的具体端到端通道：

```text
create runtime profile
  -> create or resume sandbox/computer
  -> dispatch Agent Run with profile id
  -> agent writes files in /workspace
  -> export approved artifact paths
  -> store download URL on your product run record
```

**命令行界面**

```bash
PROFILE_ID=$(miosa api POST /agent-runtime-profiles -d '{
  "workspace_id": "ws_123",
  "name": "Default app builder",
  "runtime": "claude-code",
  "tools": ["files", "exec", "previews", "artifacts"],
  "connectors": ["anthropic/workspace-claude"],
  "env": {"ANTHROPIC_MODEL": "claude-sonnet-4.6"},
  "is_default": true
}' | jq -r '.data.id')

SANDBOX_ID=$(miosa create builder-box \
  --template nextjs \
  --wait \
  --json | jq -r '.id // .data.id')

miosa prompt \
  --sandbox "$SANDBOX_ID" \
  --harness claude-code \
  "Build a lead-capture page, run tests, and write the final bundle to /workspace/dist."

miosa files cp -r "$SANDBOX_ID":/workspace/dist ./lead-capture --json
```

**打字稿**

```typescript
const profile = await miosa.agentRuntimeProfiles.create({
  workspaceId: "ws_123",
  name: "Default app builder",
  runtime: "claude-code",
  tools: ["files", "exec", "previews", "artifacts"],
  connectors: ["anthropic/workspace-claude"],
  env: { ANTHROPIC_MODEL: "claude-sonnet-4.6" },
  isDefault: true,
})

const sandbox = await miosa.sandboxes.create({
  templateId: "nextjs",
  agentRuntimeProfileId: profile.id,
})

const run = await miosa.agentRuns.run({
  sandboxId: sandbox.id,
  agentRuntimeProfileId: profile.id,
  provider: "claude-code",
  cwd: "/workspace",
  prompt:
    "Build a lead-capture page, run tests, and write the final bundle to /workspace/dist.",
  metadata: { productRunId: "run_123" },
})

const exportInfo = await sandbox.createExport({
  paths: ["/workspace/dist"],
  label: "lead-capture-build",
})

await saveProductRunArtifact(run.id, exportInfo.archiveDownloadUrl)
```

**Python**

```python
profile = client.agent_runtime_profiles.create(
    workspace_id="ws_123",
    name="Default app builder",
    runtime="claude-code",
    tools=["files", "exec", "previews", "artifacts"],
    connectors=["anthropic/workspace-claude"],
    env={"ANTHROPIC_MODEL": "claude-sonnet-4.6"},
    is_default=True,
)

sandbox = client.sandboxes.create(
    template_id="nextjs",
    agent_runtime_profile_id=profile["id"],
)

run = client.agent_runs.run(
    sandbox_id=sandbox.id,
    agent_runtime_profile_id=profile["id"],
    provider="claude-code",
    cwd="/workspace",
    prompt=(
        "Build a lead-capture page, run tests, and write the final bundle "
        "to /workspace/dist."
    ),
    metadata={"product_run_id": "run_123"},
)

export_info = sandbox.create_export(
    ["/workspace/dist"],
    label="lead-capture-build",
)

save_product_run_artifact(run["id"], export_info["archive_download_url"])
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/agent-runtime-profiles \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace_id": "ws_123",
    "name": "Default app builder",
    "runtime": "claude-code",
    "tools": ["files", "exec", "previews", "artifacts"],
    "connectors": ["anthropic/workspace-claude"],
    "is_default": true
  }'

curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"template_id":"nextjs","agent_runtime_profile_id":"arp_123"}'

curl -X POST https://api.miosa.ai/api/v1/agent-runs \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "sandbox_id": "sbx_123",
    "agent_runtime_profile_id": "arp_123",
    "provider": "claude-code",
    "cwd": "/workspace",
    "prompt": "Build a lead-capture page, run tests, and write the final bundle to /workspace/dist."
  }'

curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_123/exports \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"paths":["/workspace/dist"],"label":"lead-capture-build"}'

curl "https://api.miosa.ai/api/v1/sandboxes/sbx_123/exports/download?paths[]=/workspace/dist" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  --output lead-capture.tar.gz
```

将沙箱用于生成的文件、报告、代码、预览和应用程序包。添加
仅当运行需要浏览器交互、屏幕截图或
视觉质量检查。使用 [Files、exec 和 artifacts](/docs/develop/files-and-exec) 来
将批准的路径推广到下载中，以及[智能体运行时配置文件](/docs/api-reference/agent-runtime-profiles)
用于可重复使用的配置文件合同。

---

## 提示设备流程

无论用户是在构建应用程序、生成应用程序，还是使用一种产品级运行模型
报告，或要求操作员完成业务任务。

```text
prompt
  -> resolve workspace and agent
  -> select runtime profile
  -> select sandbox, computer, or BYOC/OpenComputer target
  -> inject scoped env and secrets
  -> stream events while the device works
  -> save files, screenshots, artifacts, preview URLs, and deploy outputs
```

对于 Lovable、Replit、Genspark、Emergent 或 Blackbox AI 风格的应用程序构建器，默认目标应该是
持久沙箱。智能体将文件写入 `/workspace`，运行安装并
在那里构建命令，启动预览服务器，修复错误，导出
可交付成果，并且仅在用户或浏览器 QA 批准后才发布
结果。

对于 Polsia 或 coFounder.co 风格的公司运营商来说，相同的流程从
业务任务而不是应用程序提示。编排器可以使用沙箱来
文档和代码、用于浏览器工作的计算机以及 BYOC/OpenComputer（当
任务必须在客户拥有的基础设施上运行。

## 运行时配置文件和 env 继承

运行时配置文件使智能体运行框架与设备保持分离。简介
答案：哪个运行时启动，允许使用哪些工具，哪些非秘密环境
存在默认值，并且可以请求哪些连接器或秘密组。

推荐的解决顺序：

|来源 |目的|
|---|---|
|租户默认配置文件 |新工作区的平台范围后备方案 |
|工作区默认配置文件 |客户/公司特定的运行时选择 |
|智能体简介 |特定于角色的工具、模型、说明和连接器 |
|运行覆盖 |一项任务的提示、cwd、超时和非秘密环境覆盖 |

环境和秘密应该狭义继承：

|值类型|继承规则|
|---|---|
|非秘密环境 |租户默认值 -> 工作区默认值 -> 配置文件环境 -> 运行环境 |
| MIOSA 运行时令牌 |每次运行或会话时使用工作区/用户/运行归因创建 |
|提供商的秘密 |通过托管机密或连接器解决，从未复制到聊天中 |
|浏览器可见的配置 |仅短暂预览/共享令牌和公共配置 |
| BYOC/OpenComputer 主机秘密 |除非明确智能体，否则留在客户拥有的主机上 |

如果某个值可以花钱、访问客户数据、向外部发布或调用
提供者 API，将其视为秘密或连接器授权而不是普通配置文件
环境。

## 工件和下载

每个智能体运行都应以命名输出结束，而不仅仅是终端文本。使用
用户应下载并存储返回的描述符的文件导出
您的产品的运行、任务或项目记录。

|输出|生产它的设备 |面向用户的表面|
|---|---|---|
|生成的应用程序 |沙盒|预览 URL、部署 URL、源 ZIP |
| PDF/DOCX/报告 |沙盒|带下载按钮的产物卡|
|截图集|电脑| QA 证据和可下载图像 |
|研究包|沙盒| Markdown/CSV/ZIP 产物 |
|客户本地结果 | BYOC/OpenComputers |主机端文件加上智能体导出（如果获得批准）|

请参阅 [文件、exec 和工件 ](/docs/develop/files-and-exec) 进行导出
API 形状。

> **注意：相关**
>
> [构建智能体公司平台](/docs/guides/build-agent-company)涵盖了该通道插入的编排器、运行时和设备模块，而[智能体编排](/docs/guides/agent-orchestration)涵盖了同时运行其中许多模块。
