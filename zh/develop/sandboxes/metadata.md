---
slug: "develop/sandboxes/metadata"
title: "元数据和标签"
description: "在创建时将任意元数据附加到沙箱，读回它，并使用外部 ID 来查询或计费所需的任何内容。"
section: "Sandboxes"
order: 2010
---

每个沙箱记录都带有一个您在创建时提供的 `metadata` 对象。它存储在记录中并在读取时返回。它是自由格式的：MIOSA 存储它并且从不解释它。

## 在创建时设置元数据

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    metadata={"team": "growth", "ticket": "ENG-482", "run": "nightly"},
)
print(sandbox.metadata)   # {"team": "growth", "ticket": "ENG-482", "run": "nightly"}
```

**打字稿**

```typescript
const sandbox = await client.sandboxes.create({
  templateId: 'miosa-sandbox',
  metadata: { team: 'growth', ticket: 'ENG-482', run: 'nightly' },
});
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"template_id": "miosa-sandbox", "metadata": {"team": "growth", "ticket": "ENG-482"}}'
```

CLI 使用重复的 `--meta` 标志设置相同的字段：

```bash
miosa create my-box --meta team=growth --meta ticket=ENG-482
```

## 元数据还是外部 ID？

这是两个不同的事情，选择很重要：

|领域|可查询吗？ |用它来 |
|---|---|---|
| `metadata` |读取时返回，而不是列表过滤器 |您随记录读回的注释、标签和指针 |
| `external_user_id`、`external_workspace_id`、`external_project_id` |是：列表和使用过滤器 |您需要列出、分组或计费的任何内容 |

`GET /api/v1/sandboxes` 按 `workspace_id`、`project_id`、`external_workspace_id`、`external_user_id`、`external_project_id` 和 `state` 进行过滤。它不会按 `metadata` 进行过滤。如果您需要枚举某个客户的每个沙箱，请使用外部 ID，而不是元数据密钥。

> **提示：归因流向下游**
>
> 三个 `external_*` 字段自动传播到每个使用记录、审核事件和派生资源（预览、部署），这正是每个租户退款所需的。元数据没有。请参阅[所有权和归属](/docs/platform/attribution)。

## 每租户归因

对于每租户产品，在创建时标记外部 ID 并在每条记录上读回它们：

```python
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    external_workspace_id="acme-corp",
    external_user_id="user-42",
    external_project_id="q3-dashboard",
    metadata={"requested_by": "jane@acme.example"},
)
```

[多租户 SaaS](/docs/cookbook/multi-tenant-saas) 中端到端涵盖了这两个领域。
