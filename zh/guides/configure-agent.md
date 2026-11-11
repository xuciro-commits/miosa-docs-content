---
slug: "guides/configure-agent"
title: "配置智能体"
description: "配置座席的身份、指令、模型、设备、工具、内存、连接帐户、触发器和审批策略。"
section: "Agents"
order: 6005
---

智能体是指具有指令、工具、设备、内存、连接的指定工作人员
账目、审批规则。目标是让智能体具有足够的可预测性
值得信任，但有足够的能力在 MIOSA 沙箱中进行实际工作，
电脑。

---

## 配置清单

|设置|为什么这很重要 |
|---|---|
|名称和描述 |帮助用户和协调者将工作分配给正确的智能体 |
|能见度|私有、工作区或平台管理 |
|说明 |角色、规则、输出样式、安全限制和域上下文 |
|模型或运行时间 |哪个模型、运行框架或自定义运行时为循环提供动力 |
|默认设备 |默认情况下工作发生的位置：沙箱、计算机或本地设备 |
|工具|文件、执行、浏览器、工件、部署、应用程序操作 |
|关联帐户 | OAuth 授权、托管连接器、API 密钥、作用域变量 |
|内存|智能体可以跨线程或作业重用持久事实 |
|触发器|计划、网络钩子、应用程序事件或用户操作 |
|审批政策|需要人工确认的操作 |
|配额|支出、子设备、运行时持续时间和并发限制 |

---

## 身份

使用有针对性的名称和描述。

好：

```text
Name: Landing Page Builder
Description: Builds, previews, and publishes branded Next.js landing pages.
```

弱：

```text
Name: Helper
Description: Does stuff.
```

当智能体的工作具体时，他们更容易委派给他们。

---

## 可见性

|模式|谁可以使用它 |最适合 |
|---|---|---|
|私人|仅限创作者 |个人实验，敏感研究|
|工作空间 |具有访问权限的会员 |共享运营商、支持、营销、开发|
|平台管理|由您的产品控制 |白标 SaaS 中的内置智能体 |

工作空间座席永远不应该默默地继承工作空间中的每一个个人联系
工作区。仅附加他们需要的帐户和工具。

---

## 说明

说明应包括：

- 智能体人的角色
- 预期输出
- 工作必须在哪里进行
- 它可能使用什么工具
- 什么需要批准
- 什么算作完成

示例：

```text
You are a Next.js build agent.
Work only inside the assigned MIOSA sandbox at /workspace.
Write files directly into the sandbox.
Run npm install, npm run lint, and npm run build before returning.
Expose port 3000 for preview.
Create artifacts for screenshots and generated reports.
Ask for approval before publishing or creating extra child sandboxes.
```

> **提示：使运行时显式**
>
> 如果智能体应该在沙箱内编码，请直接说出来。这可以防止
> 智能体在本地构建然后尝试上传完成的不良模式
> 事后回购。

---

## 模型和运行时间

MIOSA 可以支持多种运行时形状：

|运行时|用它来 |
|---|---|
| OSA | MIOSA-本机智能体/运行时操作 |
|法典|代码生成和存储库维护工作流程 |
|克劳德·代码 |使用工具和 MCP 进行智能体编码循环 |
|圆周率 |兼容的基于运行框架的智能体执行 |
|爱马仕 |自定义或供应商提供的智能体运行时 |
|定制|自带循环、队列、提示格式或模型提供程序 |

运行时应该是一个配置值，而不是硬编码到产品逻辑中。
您的产品可以为每个智能体类型选择默认运行时，并且仍然允许
高级用户稍后提供自定义运行时。

### 运行时配置文件

对于白标产品，在租户或工作区配置默认运行时
级别，而不是为每个沙箱重复相同的连接器和环境设置。

```bash
miosa api POST /agent-runtime-profiles -d '{
  "name": "Default app builder",
  "runtime": "claude-code",
  "tools": ["files", "exec", "previews", "artifacts"],
  "connectors": ["refero/design-research", "anthropic/workspace-claude"],
  "policy": {"approval": "confirm-risky", "max_child_sandboxes": 5},
  "is_default": true
}'
```

```typescript
await miosa.agentRuntimeProfiles.create({
  name: "Default app builder",
  runtime: "claude-code",
  tools: ["files", "exec", "previews", "artifacts"],
  connectors: ["refero/design-research", "anthropic/workspace-claude"],
  policy: { approval: "confirm-risky", max_child_sandboxes: 5 },
  isDefault: true,
})
```

```python
client.agent_runtime_profiles.create(
    name="Default app builder",
    runtime="claude-code",
    tools=["files", "exec", "previews", "artifacts"],
    connectors=["refero/design-research", "anthropic/workspace-claude"],
    policy={"approval": "confirm-risky", "max_child_sandboxes": 5},
    isDefault=True,
)
```

使用配置文件作为默认值。使用 `miosa prompt` 或智能体运行 API
个人任务调度。

当默认配置文件存在时，该配置文件中的每个新沙箱或云计算机
租户/工作区会自动获取配置文件，除非调用者选择退出。
MIOSA 在插入运行时之前应用配置文件：

|简介领域 |沙箱行为 |计算机行为|
|---|---|---|
| `runtime` |存储在沙箱元数据中，以便编排器知道要启动哪个智能体工具 |存储在计算机设置中以进行编排和调度 |
| `tools` |作为预期的工具包存储在元数据中 |作为预期的工具包存储在设置中 |
| `env` |在启动时合并到沙箱环境中；请求级环境获胜 |在计算机环境存储中加密并在 OSA 正常后同步 |
| `connectors` |通过 MIOSA 出口作为不透明占位符令牌进行绑定 |通过 MIOSA 出口作为不透明占位符令牌进行绑定 |
| `policy` |存储在元数据中以供批准、子设备和配额决策|存储在审批、子设备和配额决策的设置中 |

```bash
# Uses the workspace or tenant default profile automatically.
miosa create builder --template nextjs --ttl 1h --wait

# Force a specific profile for this sandbox (API):
miosa api POST /sandboxes -d '{
  "name": "builder",
  "template": "nextjs",
  "agent_runtime_profile_id": "9f4b3d8e-0000-0000-0000-000000000000"
}'

# Create a browser/desktop computer with the workspace default profile.
miosa computer create browser-worker
```

```typescript
const sandbox = await miosa.sandboxes.create({
  templateId: "nextjs",
  agentRuntimeProfileId: "9f4b3d8e-0000-0000-0000-000000000000",
})

const computer = await miosa.computers.create({
  name: "browser-worker",
  agentRuntimeProfileId: "9f4b3d8e-0000-0000-0000-000000000000",
})
```

```python
sandbox = client.sandboxes.create(
    template_id="nextjs",
    agent_runtime_profile_id="9f4b3d8e-0000-0000-0000-000000000000",
)

computer = client.computers.create(
    name="browser-worker",
    agent_runtime_profile_id="9f4b3d8e-0000-0000-0000-000000000000",
)
```

连接器机密不会复制到元数据中或返回给智能体。的
运行时看到环境变量，例如 `REFERO_MCP_TOKEN` 或 `ANTHROPIC_API_KEY`，但是
这些值是 MIOSA 出口占位符 (`miosa-tok-*`)。智能体交换
仅当请求被允许时，提供者凭证的占位符。

---

## 默认设备

每个智能体都应该有一个默认的设备策略。

|政策 |行为 |
|---|---|
|沙盒优先 |创建或恢复代码、文件、命令、预览和工件的沙箱 |
|计算机优先 |使用持久桌面/浏览器计算机执行 GUI 和 Web 任务 |
|本地优先 |当需要本地文件或专用网络访问时，使用配对的用户计算机 |
|协调员|从一个根设备启动并启动子设备以进行专门工作 |

对于应用程序构建者来说，沙箱优先应该是默认的。对于浏览器和 RPA
智能体，计算机优先通常是正确的。

---

## 工具

仅附加智能体所需的工具包。

常见捆绑包：

|捆绑 |包括 |
|---|---|
|构建应用程序 |文件、执行、实时预览、快照、工件、发布 |
|浏览器质量检查 |浏览器、屏幕截图、点击、表单填写、预览访问 |
|研究|网页搜索、页面提取、Refero 设计研究、笔记文件 |
|产物工厂|文件、执行、媒体转换、工件、版本历史记录 |
|集成|连接帐户、网络钩子、应用程序操作 |
|操作员|计算机、浏览器、终端、长时间运行的作业 |

---

## 账户和变量

使用帐户连接和变量而不是聊天粘贴的秘密。

|类型 |示例|如何曝光 |
|---|---|---|
| OAuth 帐户 | GitHub、谷歌、Slack |短期令牌或出口经纪占位符 |
| API 密钥 | Anthropic、OpenAI、Stripe |范围环境变量 |
|托管连接器| Refero设计研究| MIOSA 拥有的代币，仅中介给提供商 |
|租户变量 | `NEXT_PUBLIC_BRAND_NAME` |普通环境变量 |
|运行时令牌 | `MIOSA_API_KEY` |一个工作区、用户、项目或运行的范围令牌 |

切勿将秘密粘贴到智能体对话中。通过仪表板存储它，
API、CLI 或连接器流程。

---

## 内存

内存是智能体可以重复使用的持久上下文。保持其范围。

美好回忆：

- 品牌颜色
- 项目路线
- 客户偏好
- 部署 URL
- 应用架构笔记
- 重复任务规则

避免保存：

- 原始秘密
- 一次性 OTP
- 不相关的聊天记录
- 陈旧的调试猜测

工作区所有者应该可以编辑和撤销内存。

---

## 触发器和作业

智能体可以手动或自动运行。

|触发|示例|
|---|---|
|手册|用户要求智能体构建登陆页面 |
|日程 |每日进行竞争对手抓取 |
|网络钩子 |新的 GitHub 问题创建了分类任务 |
|应用事件 |新的 CRM 主管生成提案 |
|运行时事件 |失败的部署创建修复任务 |

触发的智能体应写入运行记录、流式传输事件并遵守配额。

---

## 批准政策

危险行为需要批准：

- 发布到生产
- 创建超过阈值的额外子设备
- 向外部发送消息
- 写入连接的应用程序
- 超出配置预算的支出积分
- 删除文件、快照、部署或数据
- 使用高权限凭证

在分配的工作空间内进行例行读取、文件写入、测试和预览
启动通常可以是自动的。

---

## 相关

- [智能体可以做什么](/docs/guides/agent-capabilities)：工具目录、权限模型和产品界面。
- [智能体设备](/docs/guides/agent-devices)：为代码、浏览器、桌面、本地网络或部署选择正确的运行时。
- [智能体编排](/docs/guides/agent-orchestration)：协调多个智能体、队列、设备池和子智能体。
- [托管连接器](/docs/guides/managed-connectors)：在不暴露提供商凭据的情况下提供智能体提供商工具。
