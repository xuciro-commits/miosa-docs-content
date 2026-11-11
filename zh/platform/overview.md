---
slug: "platform/overview"
title: "平台构建者指南"
description: "在 MIOSA 上运行用户的智能体、预览和工作区：架构、成本与资源配置模式。"
section: "Platform builders"
order: 9001
---

MIOSA 为你提供机器、预览和部署。
平台是你在这些能力之上构建的产品，让其他人无需了解其运作方式也能使用。

本指南面向构建 Lovable、Devin 这类产品或智能体平台的开发者。
在这类产品中，你的产品会为每一位自己的用户创建一个沙箱。

这样的平台需要：

- 为每个用户或项目提供一个沙箱：[模式](/docs/platform/patterns)。
- 用户所构建内容的私有预览 URL：[品牌与域名设置](/docs/platform/white-label)。
- 在机器内工作的智能体或 harness：[产品中的智能体](/docs/platform/agents-in-your-product)。
- 部署与 Docker Host：[部署概览](/docs/docker-host/overview)。
- 白标、租户、限定范围的令牌和外部用户：[品牌与域名设置](/docs/platform/white-label)与[所有权与归属](/docs/platform/attribution)。
- 计费与上限：[模板与规格](/docs/platform/templates-and-sizing)。

## 选择形态

MIOSA 上的每个平台都采用同一种模型。
三种形态覆盖了大多数产品。

| 你的身份 | 示例 | 使用者 | 白标内容 |
|---|---|---|---|
| 一家推出智能体产品的 SaaS 公司 | **Lumen** | Lumen 的客户 | 全部内容：域名、品牌、计费 |
| 一家承接客户项目的智能体机构 | **Northwind Agency** | Northwind 的客户 | 客户预览和交付域名 |
| 一家拥有团队的公司 | **Acme Corp** | Acme 自己的员工和智能体 | 无。仅供内部使用 |

如果你只需要内部场景，请直接跳到 [Acme Corp：内部平台](/docs/platform/patterns#acme-corp-an-internal-platform)。

## Northwind Agency：客户项目

完整架构——Lumen、Northwind Agency 客户项目和 Acme Corp——在[模式](/docs/platform/patterns)中逐步讲解。

## 1. 创建限定范围的平台密钥

平台密钥（`msk_p_*`）用于服务端自动化。
只有租户所有者和管理员可以创建此类密钥。
为每个密钥只授予后端所需的权限范围，并可选择将其限制在一个工作区。

**TypeScript**

```typescript
const key = await miosa.apiKeys.create({
  name: "Lumen backend prod",
  key_type: "platform",
  workspace_id: workspaceId,
  scopes: ["sandboxes:write", "deployments:write", "domains:write"],
})

console.log(key.key) // shown once, store it in your secret manager
```

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

key = miosa.api_keys.create(
    name="Lumen backend prod",
    key_type="platform",
    workspace_id=workspace_id,
    scopes=["deployments:write", "sandboxes:write", "domains:write"],
)

print(key["key"])  # shown once
```

明文只返回一次，MIOSA 仅存储哈希。
轮换、权限范围和审计请参见 [API 密钥](/docs/platform/api-keys)。

## 2. 为每个客户创建工作区

在客户注册时执行一次，并将返回的 ID 与你自己的记录一起保存。

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const workspace = await miosa.workspaces.create({
  name: "Northwind Agency",
  slug: "northwind-agency",
  externalWorkspaceId: "acct_123",
})
```

**Python**

```python
import miosa, os

client = miosa.Miosa(api_key=os.environ["MIOSA_API_KEY"])

workspace = client.workspaces.create(
    name="Northwind Agency",
    slug="northwind-agency",
    external_workspace_id="acct_123",
)
```

你也可以跳过这一步。
如果创建调用携带 `workspace_slug` 或 `external_workspace_id`，而该工作区不存在，MIOSA 会解析或创建它。
响应会返回规范的 `workspace_id` 和 `project_id`，请保存并重复使用。
详情见[所有权与归属](/docs/platform/attribution)。

## 3. 为每个用户提供一台机器

每个最终用户一个沙箱或计算机，绝不在客户之间共享。

**TypeScript**

```typescript
const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  workspaceId: workspace.id,
  projectSlug: "booking-portal",
  projectName: "Booking Portal",
  externalUserId: user.id,         // your end user
  externalWorkspaceId: "acct_123", // your customer
})
```

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    workspace_id=workspace.id,
    project_slug="booking-portal",
    project_name="Booking Portal",
    external_user_id=user.id,
    external_workspace_id="acct_123",
)

# Everything in one customer's workspace
mine = client.sandboxes.list(workspace_id=sandbox.workspace_id)
```

每台机器都有自己的文件系统、网络和进程树。
规格、模板和空闲默认值见[模板与规格](/docs/platform/templates-and-sizing)。

## 按需选择

| 你想要 | 页面 |
|---|---|
| 为每个最终用户配置一个沙箱 | [模式](/docs/platform/patterns) |
| 在用户的机器内运行智能体 | [产品中的智能体](/docs/platform/agents-in-your-product) |
| 选择规格、模板和空闲默认值 | [模板与规格](/docs/platform/templates-and-sizing) |
| 把品牌应用到预览和应用上 | [品牌与域名设置](/docs/platform/white-label) |
| 将资源和使用量归属到你的客户 | [所有权与归属](/docs/platform/attribution) |
| 计量用量并向客户收费 | [用量与计费](/docs/platform/usage-and-billing) |

## 接下来

- [模式](/docs/platform/patterns)：覆盖大多数产品的三种架构，以及如何配置每一种。
- [产品中的智能体](/docs/platform/agents-in-your-product)：为每个用户提供一台机器，并在其中运行智能体或 harness。
- [连接智能体账户](/docs/guides/connect-agent-accounts)：从你自己的代码中连接客户的 Claude 或 ChatGPT 套餐，或他们的模型密钥。
- [模板与规格](/docs/platform/templates-and-sizing)：规格、成本、模板、配额和空闲默认值。
- [内部业务操作系统](/docs/guides/internal-business-os)：本指南的企业内部版本。

## 相关

- [API 密钥](/docs/platform/api-keys)：密钥类别、权限范围、轮换和审计。
- [所有权与归属](/docs/platform/attribution)：工作区和项目的所有权，以及你的外部 ID。
- [品牌与域名设置](/docs/platform/white-label)：预览域名、部署域名、自定义域名和品牌。
- [浏览器令牌](/docs/platform/browser-tokens)：用于最终用户浏览器中预览和终端的短期令牌。
- [连接智能体账户](/docs/guides/connect-agent-accounts)：用于你的工作区或每个最终用户的密钥和订阅套餐。
- [用量与计费](/docs/platform/usage-and-billing)：在单张 MIOSA 账单上按客户统计用量并计费。
- [构建图库](/docs/build)：20 多种产品形态的分步指南。

## 参考集成

可运行的示例位于 [`examples/`](https://github.com/Miosa-osa/miosa/tree/main/examples)。
参见 [`examples/agent-builds-static-app/`](https://github.com/Miosa-osa/miosa/tree/main/examples/agent-builds-static-app)，了解“智能体构建、预览并部署”的模式。
