---
slug: "api-reference/workspaces"
title: "工作区 API"
description: "用于创建和管理 MIOSA 工作区（组织内的客户/客户端层）的 API 参考。"
section: "Organization & billing"
order: 13000
---

工作空间对组织内部的资源进行分组。在白标平台中，工作区通常代表一个下游客户或客户，例如 `Lumen` 组织内的 `Acme Corp`。项目位于工作区中，沙箱、计算机、部署、数据库、存储桶、卷、功能、作业和域都属于项目。

基本路径：`/api/v1/workspaces`

> **注意**
>
> 当您注册时，会自动创建一个**默认**工作区。除非您在创建时指定工作区/项目所有权，否则资源将分配给默认工作区及其默认项目。

---

## 快速入门
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa(); // reads MIOSA_API_KEY

// Create a workspace for a downstream customer
const ws = await client.workspaces.create({
  name: 'Acme Corp',
  slug: 'acme-corp',
  externalWorkspaceId: 'acct_123',
});

// Create a computer inside a project in that workspace
const computer = await client.computers.create({
  name: 'workspace-builder',
  template_type: 'miosa-desktop',
  workspaceId: ws.id,
  projectSlug: 'lead-magnet',
  projectName: 'Lead Magnet',
});

// List computers in the workspace
const { data } = await client.workspaces.listComputers(ws.id);
```
```bash
# curl equivalent
curl -X POST https://api.miosa.ai/api/v1/workspaces \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Acme Corp", "slug": "acme-corp", "external_workspace_id": "acct_123"}'
```

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/workspaces` |列出租户的工作区 |
| `POST` | `/workspaces` |创建工作区 |
| `GET` | `/workspaces/{id}` |获取工作空间 |
| `PATCH` | `/workspaces/{id}` |更新工作区 |
| `PUT` | `/workspaces/{id}/settings` |更新工作区级资源限制 |
| `DELETE` | `/workspaces/{id}` |删除工作区 |
| `GET` | `/workspaces/{id}/computers` |列出工作区中的计算机 |
| `GET` | `/workspaces/{id}/stats` |计算机数量和资源总量 |
| `GET` | `/workspaces/{id}/usage` |时间序列信用和计算使用情况 |
| `GET` | `/workspaces/{id}/projects` |列出工作区中的项目 |

---

## 列出工作空间

**`GET /api/v1/workspaces`**

返回属于经过身份验证的租户的所有工作区。

### 响应 - `200 OK`
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "tenant_id": "...",
      "name": "default",
      "slug": "default",
      "external_workspace_id": null,
      "description": null,
      "metadata": {},
      "is_default": true,
      "computer_count": 3,
      "created_at": "2026-04-11T00:00:00Z",
      "updated_at": "2026-04-11T00:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
curl https://api.miosa.ai/api/v1/workspaces \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 创建工作空间

**`POST /api/v1/workspaces`**

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `name` |字符串|是的 |人类可读的名称（1-120 个字符）|
| `slug` |字符串|没有 | URL 安全标识符。如果省略，则自动从 `name` 派生。模式：`[a-z0-9][a-z0-9-]{0,79}` |
| `external_workspace_id` |字符串|没有 |您的客户/帐户/工作区 ID。在这个 MIOSA 组织中，全球独一无二。 |
| `description` |字符串|没有 |可选说明|
| `metadata` |对象|没有 |存储在工作区上的调用者元数据。 |

### 响应 - `201 Created`

完整的工作区对象（与上面的列表项形状相同）。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 422 | `has already been taken`| Slug 已存在于租户中 |
| 422 | `must start with alphanumeric and contain only a-z 0-9 -`|无效的 slug 格式 |
```bash
curl -X POST https://api.miosa.ai/api/v1/workspaces \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme Corp",
    "slug": "acme-corp",
    "external_workspace_id": "acct_123",
    "description": "Client workspace for Acme Corp"
  }'
```

---

## 获取工作空间

**`GET /api/v1/workspaces/{id}`**

### 路径参数

|参数|类型 |描述 |
|------------|------|-------------|
| `id` | UUID |工作区 ID |

### 响应 - `200 OK`

完整的工作区对象。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 403 | `forbidden`|工作区属于不同的租户|
| 404 | `workspace not found`|不存在|
```bash
curl https://api.miosa.ai/api/v1/workspaces/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 更新工作区

**`PATCH /api/v1/workspaces/{id}`**

更新可变工作区字段。一旦资源和 URL 存在，最好将 slug 视为稳定的。

### 请求正文

|领域|类型 |描述 |
|--------|------|-------------|
| `name` |字符串|新显示名称 |
| `slug` |字符串|新的 URL 安全 slug。在组织内部必须保持独特。 |
| `description` |字符串|新描述（`null` 清除）|
| `external_workspace_id` |字符串|您的客户/帐户/工作空间 ID |
| `metadata` |对象|替换元数据图 |

### 响应 - `200 OK`

更新了工作区对象。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 403 | `forbidden`|错误的租客 |
| 404 | `workspace not found`|不存在|
```bash
curl -X PATCH https://api.miosa.ai/api/v1/workspaces/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "production-us-east"}'
```

---

## 删除工作区

**`DELETE /api/v1/workspaces/{id}`**

删除空工作区。如果工作区包含计算机或项目，则会失败。

### 响应 - `200 OK`
```json
{ "deleted": true }
```

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 403 | `forbidden`|错误的租客 |
| 404 | `workspace not found`|不存在|
| 409 | `workspace has computers`|首先删除或移动资源 |
```bash
curl -X DELETE https://api.miosa.ai/api/v1/workspaces/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 更新工作区设置

**`PUT /api/v1/workspaces/{id}/settings`**应用工作区的资源限制和默认首选项。这些限制限制了工作区中任何计算机可以执行的操作。租户级别限制仍然适用作为上限。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `max_computers` |整数 |没有 |此工作区允许的最大计算机数量。 `null` = 无工作区级别上限（租户限制适用）。 |
| `max_computer_size` |字符串|没有 |可以使用以下命令创建此工作区中的计算机的最大 `size` 值：`"small"`、`"medium"` 或 `"large"`。 |
| `default_computer_size` |字符串|没有 |创建没有显式 `size` 的计算机时应用默认 `size`。 |
| `default_auto_stop_seconds` |整数 |没有 |默认情况下，自动停止空闲超时（秒）应用于新计算机。 `0` = 无自动停止。 |
| `allowed_template_types` |字符串[] |没有 |可在此工作区中使用的模板名称白名单。空数组 = 允许所有模板。 |
| `monthly_spend_cap_cents` |整数 |没有 |此工作区中托管沙盒支出的每月上限（以积分为单位（1 积分 = 1 美分））。 `null` 将其清除。 |
| `monthly_total_spend_cap_cents` |整数 |没有 |每月上限（以积分为单位）针对此工作区解决的所有事项：沙箱、计算机、部署、数据库、存储、快照、备份和功能。 `null` 将其清除。 |
| `spend_cap_action` |字符串|没有 |达到上限的作用：`"pause"`（默认）使用 HTTP 402 拒绝工作区中的新资源并停止那里正在运行的内容，`"alert"` 仅发送阈值警报。 |
| `entity_spend_caps` |对象|没有 |每个 `entity` 标签的每月上限，例如 `{"customer-a": 5000}`。 |

没有什么会被破坏，也没有什么会自行恢复。该设置名为 `spend_cap_action`，而不是 `action`。

### 响应 - `200 OK`
```json
{
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "settings": {
    "max_computers": 10,
    "max_computer_size": "large",
    "default_computer_size": "small",
    "default_auto_stop_seconds": 3600,
    "allowed_template_types": []
  },
  "updated_at": "2026-05-17T10:00:00Z"
}
```

---

## 域名

工作区不拥有预览域。预览域是一个组织
设置和每个部署自定义域附加到部署：

- [自定义域 API](/docs/api-reference/custom-domains) - 组织预览域和部署/计算机自定义域。

---

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 403 | `FORBIDDEN`|调用者不是组织所有者或管理员 (`Only workspace admins may update workspace limits`) |
| 403 | `forbidden`|错误的租客 |
| 404 | `workspace not found`|不存在|
| 422 | `VALIDATION_FAILED`|大小值未知或限制无效 |
| 422 | `UNKNOWN_WORKSPACE_SETTING`| API 不知道的设置名称。 `message` 命名最接近的接受键（发送 `action` 回答 `Did you mean "spend_cap_action"?`），`allowed` 列出每个接受的设置。 |
```bash
curl -X PUT https://api.miosa.ai/api/v1/workspaces/{id}/settings \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "max_computers": 10,
    "max_computer_size": "large",
    "default_auto_stop_seconds": 3600
  }'
```

---

## 列出工作区中的计算机

**`GET /api/v1/workspaces/{id}/computers`**

### 响应 - `200 OK`
```json
{
  "data": [
    {
      "id": "...",
      "name": "runner-1",
      "status": "running",
      "size": "small",
      "template_type": "miosa-desktop",
      "created_at": "2026-04-11T00:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
curl https://api.miosa.ai/api/v1/workspaces/{id}/computers \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 列出工作区中的项目

**`GET /api/v1/workspaces/{id}/projects`**

返回工作区拥有的项目。在列出项目内的沙箱、计算机、部署和数据库之前，将其用于客户端仪表板。

### 响应 - `200 OK`
```json
{
  "data": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "tenant_id": "...",
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "external_workspace_id": "acct_123",
      "external_project_id": "project_789",
      "name": "Lead Magnet",
      "slug": "lead-magnet",
      "description": null,
      "metadata": {},
      "created_at": "2026-05-18T10:00:00Z",
      "updated_at": "2026-05-18T10:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
curl https://api.miosa.ai/api/v1/workspaces/{id}/projects \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取工作空间统计信息

**`GET /api/v1/workspaces/{id}/stats`**

返回按状态分组的计算机计数快照以及工作区的聚合资源占用量。

### 响应 - `200 OK`
```json
{
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "computers": {
    "total": 5,
    "by_status": {
      "running": 3,
      "stopped": 2,
      "provisioning": 0,
      "error": 0
    }
  },
  "resources": {
    "total_vcpus": 7,
    "total_memory_mb": 28672,
    "total_disk_gb": 160
  },
  "settings": {
    "max_computers": 10,
    "max_computer_size": "large",
    "default_auto_stop_seconds": 3600
  }
}
```
```bash
curl https://api.miosa.ai/api/v1/workspaces/{id}/stats \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取工作空间使用情况

**`GET /api/v1/workspaces/{id}/usage`**

返回给定窗口内工作区的时间序列信用消耗和计算小时总计。将其用于每个工作空间的成本归因仪表板。

### 查询参数

|参数|类型 |描述 |
|------------|------|-------------|
| `window` |字符串| `"24h"`（默认）、`"7d"`、`"30d"` |
| `granularity` |字符串| `"hour"`（默认），`"day"` - 时间序列数据的存储桶大小 |

### 响应 - `200 OK`
```json
{
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "window": "7d",
  "granularity": "day",
  "total_credits_used": 1284,
  "total_compute_hours": 42.5,
  "series": [
    {
      "ts": "2026-05-10T00:00:00Z",
      "credits_used": 198,
      "compute_hours": 6.6
    },
    {
      "ts": "2026-05-11T00:00:00Z",
      "credits_used": 211,
      "compute_hours": 7.0
    }
  ]
}
```

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 403 | `forbidden`|错误的租客 |
| 404 | `workspace not found`|不存在|
| 422 | `VALIDATION_FAILED`|未知的 `window` 或 `granularity` 值 |
```bash
curl "https://api.miosa.ai/api/v1/workspaces/{id}/usage?window=7d&granularity=day" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 常见食谱

### 将所有计算机移至新工作区计算机尚不支持通过 `PATCH /computers/{id}` 对工作区分配进行 `move` 操作。请改用 `POST /computers/{id}/move` 端点：
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{computer_id}/move \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workspace_id": "target-workspace-id"}'
```

### 枚举所有工作区及其大小
```typescript
const { data: workspaces } = await client.workspaces.list();
for (const ws of workspaces) {
  console.log(`${ws.name}: ${ws.computerCount} computers`);
}
```

### 默认工作区 ID 查找

如果您需要以编程方式使用默认工作区 ID：
```typescript
const { data } = await client.workspaces.list();
const defaultWs = data.find(ws => ws.isDefault);
console.log(defaultWs.id);
```

---

## 常见错误

|状态 |代码|原因 |
|--------|------|--------|
| 403 | `forbidden`|工作区属于不同的租户|
| 404 | `workspace not found`|不存在|
| 409 | `workspace has computers`|无法删除包含计算机的工作区 |
| 422 | `VALIDATION_FAILED`|名称或 slug 架构验证失败 |

---

## 另请参阅

- [计算机 API](/docs/api-reference/computers) - 计算机的范围仅限于工作区
- [Projects API](/docs/api-reference/projects) - 项目位于工作区中
- [错误代码](/docs/api-reference/errors) - `VALIDATION_FAILED` 和一般错误
