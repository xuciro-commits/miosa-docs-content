---
slug: "api-reference/projects"
title: "项目API"
description: "用于在 MIOSA 工作区中创建和管理项目的 API 参考。"
section: "API reference"
order: 12044
---

**项目**是工作区中的应用程序、网站、磁铁、文档、工作流程或客户构建。项目拥有沙箱、计算机、部署、数据库、存储桶、卷、功能、作业、身份验证配置、集成和自定义域。

基本路径：`/api/v1/projects`

> **注意**
>
> 如果资源创建调用包括 `project_slug`、`project_name` 或 `external_project_id`，MIOSA 将在创建资源之前解析或创建项目。您还可以使用此 API 显式创建项目。

## 端点

|方法|路径|描述 |
|---|---|---|
| `GET` | `/projects` |列出组织的项目 |
| `POST` | `/projects` |在工作区中创建项目 |
| `GET` | `/projects/{id}` |获取一个项目 |
| `PATCH` | `/projects/{id}` |更新项目 |
| `GET` | `/workspaces/{id}/projects` |在一个工作区中列出项目 |

---

## 列出项目

**`GET /api/v1/projects`**

### 查询参数

|参数|类型 |描述 |
|---|---|---|
| `workspace_id` | UUID |将结果限制在一个工作区 |

### 响应 - `200 OK`
```json
{
  "data": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "tenant_id": "tnt_abc123",
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "external_workspace_id": "acct_123",
      "external_project_id": "project_789",
      "name": "Lead Magnet",
      "slug": "lead-magnet",
      "description": null,
      "metadata": {},
      "settings": {},
      "created_at": "2026-05-18T10:00:00Z",
      "updated_at": "2026-05-18T10:00:00Z"
    }
  ],
  "total": 1
}
```

```bash
curl "https://api.miosa.ai/api/v1/projects?workspace_id=550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 创建一个项目

**`POST /api/v1/projects`**

项目必须属于工作区。提供 `workspace_id`、`workspace_slug` 或 `external_workspace_id` 之一。

### 请求正文

|领域|类型 |必填 |描述 |
|---|---|---|---|
| `workspace_id` | UUID |有条件|现有 MIOSA 工作区 ID |
| `workspace_slug` |字符串|有条件|现有工作区 slug |
| `external_workspace_id` |字符串|有条件|您的外部工作区/客户 ID |
| `name` |字符串|是的 |人类可读的项目名称 |
| `slug` |字符串|没有 | URL 安全标识符。如果省略，则自动从 `name` 派生。 |
| `external_project_id` |字符串|没有 |您的项目/应用程序/文档 ID |
| `description` |字符串|没有 |可选说明|
| `metadata` |对象|没有 |存储在项目中的调用者元数据 |

### 示例
```bash
curl -X POST https://api.miosa.ai/api/v1/projects \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace_slug": "acme-corp",
    "name": "Lead Magnet",
    "slug": "lead-magnet",
    "external_workspace_id": "acct_123",
    "external_project_id": "project_789"
  }'
```

### 响应 - `201 Created`

完整的项目对象。

### 错误

|状态 |代码|原因 |
|---|---|---|
| 400 | `WORKSPACE_REQUIRED` |未提供 `workspace_id`、`workspace_slug` 或 `external_workspace_id` |
| 404 | `WORKSPACE_NOT_FOUND`|工作区选择器在组织内部未解析 |
| 422 | `VALIDATION_FAILED`|名称、slug 或外部项目 ID 验证失败 |

---

## 获取项目

**`GET /api/v1/projects/{id}`**

如果项目属于经过身份验证的组织，则返回一个项目。

```bash
curl https://api.miosa.ai/api/v1/projects/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 更新项目

**`PATCH /api/v1/projects/{id}`**

### 请求正文

|领域|类型 |描述 |
|---|---|---|
| `name` |字符串|新显示名称 |
| `slug` |字符串|新的 URL 安全 slug，在工作区中独一无二 |
| `description` |字符串|新描述（`null` 清除）|
| `external_project_id` |字符串|您的项目/应用程序/文档 ID |
| `external_workspace_id` |字符串|您的客户/工作区 ID |
| `metadata` |对象|替换元数据图 |
| `settings` |对象|更换设置图。 |

```bash
curl -X PATCH https://api.miosa.ai/api/v1/projects/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Summer Lead Magnet"}'
```

---

## 在资源创建上使用项目

大多数创建端点接受相同的所有权字段：

```json
{
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "project_id": "660e8400-e29b-41d4-a716-446655440001",
  "external_workspace_id": "acct_123",
  "external_project_id": "project_789"
}
```

您可以使用 slugs 代替 UUID：

```json
{
  "workspace_slug": "acme-corp",
  "workspace_name": "Acme Corp",
  "project_slug": "lead-magnet",
  "project_name": "Lead Magnet"
}
```

这适用于沙箱、计算机、部署、数据库、存储桶、卷、边缘功能、cron 作业、项目身份验证和集成。派生记录自动继承所有权。

---

## 域名

项目不拥有预览域。预览域是一个组织
设置和每个部署自定义域附加到部署：

- [自定义域 API](/docs/api-reference/custom-domains) - 组织预览域和部署/计算机自定义域。

## 另请参阅

- [Workspaces API](/docs/api-reference/workspaces) - 父资源
- [所有权和归属](/docs/platform/attribution) - ID 如何流向使用和事件
- [Sandboxes API](/docs/api-reference/sandboxes) - 在项目内创建资源
- [部署 API](/docs/api-reference/deployments) - 将项目发布到 URL
