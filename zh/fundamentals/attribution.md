---
slug: "fundamentals/attribution"
title: "所有权和外部归属"
description: "规范的所有权字段、不透明的外部归属以及将 MIOSA 转售给自己的客户的平台的使用情况汇总。"
section: "API reference"
order: 12005
---

每个 MIOSA 资源都接受规范的所有权字段和可选的归属字段：

|领域|意义|
|---|---|
| `workspace_id` / `workspace_slug` |拥有该资源的 MIOSA 工作区 |
| `project_id` / `project_slug` |拥有该资源的MIOSA项目|
| `external_workspace_id` |您客户的帐户/租户 ID |
| `external_user_id` |该帐户内的最终用户 |
| `external_project_id` |您自己的数据库中的项目或应用 ID |

每个资源还接受自由格式的 `metadata` 对象（最多 16 个键值字符串对）。使用它来承载应用程序层上下文 - 功能标志、计划层、区域，无论您的计费系统需要什么 - 而不会通过 MIOSA 资源连接使您自己的数据库膨胀。

集成商将这些字段用于四个主要目的：

- **使用情况汇总** - 查询 `GET /api/v1/sandboxes?external_workspace_id=acme-co` 以汇总下游客户的计算时间和成本。
- **每最终用户配额** - 在创建新沙箱之前，通过计算每个 `external_user_id` 的活动沙箱来强制实施您自己的速率限制。
- **审核日志过滤** - 审核日志接受 `external_user_id` 和 `external_project_id` 作为过滤器，以便您可以提取特定于客户的活动跟踪，而不会暴露其他客户的数据。
- **退款和计费** - `external_workspace_id` 是当您向下游客户转售 MIOSA 计算时建议用于计费汇总的键。请参阅[审核日志](/docs/api-reference/audit-log)和[配额](/docs/api-reference/quotas)参考。

MIOSA 在每个资源上存储规范的 `workspace_id` 和 `project_id`。外部字段存储为不透明字符串。两者都出现在列表/读取响应中，并且列表端点接受它们作为过滤器：

**Python**

```python
# Create a sandbox attributed to a specific customer
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    workspace_slug="acme-co",
    workspace_name="Acme Co",
    project_slug="invoice-bot",
    project_name="Invoice Bot",
    external_workspace_id="acme-co",
    external_user_id="alice@example.net",
    external_project_id="invoice-bot",
    metadata={"plan": "pro", "region": "us-east"},
)

# List only sandboxes for that customer
sandboxes = client.sandboxes.list(
    workspace_id=sandbox.workspace_id,
)
```

**打字稿**

```typescript
// Create
const sandbox = await miosa.sandboxes.create({
  templateId: 'miosa-sandbox',
  workspaceSlug: 'acme-co',
  workspaceName: 'Acme Co',
  projectSlug: 'invoice-bot',
  projectName: 'Invoice Bot',
  externalWorkspaceId: 'acme-co',
  externalUserId: 'alice@example.net',
  externalProjectId: 'invoice-bot',
  metadata: { plan: 'pro', region: 'us-east' },
});

// Filter
const sandboxes = await miosa.sandboxes.list({
  workspaceId: sandbox.workspaceId,
});
```

**curl**

```bash
# Create
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "template_id": "miosa-sandbox",
    "workspace_slug": "acme-co",
    "workspace_name": "Acme Co",
    "project_slug": "invoice-bot",
    "project_name": "Invoice Bot",
    "external_workspace_id": "acme-co",
    "external_user_id": "alice@example.net",
    "external_project_id": "invoice-bot",
    "metadata": { "plan": "pro", "region": "us-east" }
  }'

# Filter
curl "https://api.miosa.ai/api/v1/sandboxes?workspace_id=550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer msk_u_..."
```

过滤范围始终限于您的 MIOSA 组织。无论您传递什么工作区、项目或外部 ID，您都无法看到其他组织的资源。

请参阅[所有权和归属](/docs/platform/attribution)了解完整的过滤器规范和按工作区/项目划分的计费。

- [所有权和归属](/docs/platform/attribution)：白标平台如何界定客户、项目和资源。
