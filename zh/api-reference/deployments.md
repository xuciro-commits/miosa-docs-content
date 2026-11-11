---
slug: "api-reference/deployments"
title: "API 参考/部署"
description: "用于部署资源的 REST API - 创建、列出、获取、更新、删除、发布、回滚。"
section: "API reference"
order: 12038
---

**部署**是已发布的应用程序/站点/API 的稳定生产对象。请参阅[部署/概述](/docs/docker-host/overview)了解概念模型。

> **注意：推荐的发布路径**
>
> 使用 `POST /api/v1/sandboxes/:id/publish` 或 `miosa deploy create` 将经过测试的沙箱转变为不可变版本。
> 更新时传递现有部署 ID，以便其身份和 URL 保持稳定。

## 端点
```http
POST   /api/v1/deployments
GET    /api/v1/deployments
GET    /api/v1/deployments/:id
PATCH  /api/v1/deployments/:id
DELETE /api/v1/deployments/:id
POST   /api/v1/deployments/:id/publish
POST   /api/v1/deployments/:id/rollback
POST   /api/v1/sandboxes/:id/publish
POST   /api/v1/sandboxes/:id/deploy                    # Compatibility bridge
```

所有端点都需要 `Authorization: Bearer &lt;msk_*&gt;`。突变应包含 `Idempotency-Key` 标头。

## 创建
```http
POST /api/v1/deployments
```

身体：
```json
{
  "name": "Team Portal",
  "source_type": "sandbox",
  "workspace_slug": "acme-corp",
  "workspace_name": "Acme Corp",
  "project_slug": "team-portal",
  "project_name": "Team Portal",
  "external_workspace_id": "acct_123",
  "external_user_id": "user-456",
  "external_project_id": "project_789",
  "metadata": { }
}
```

创建时接受的所有权字段：`workspace_id`、`workspace_slug`、`workspace_name`、`project_id`、`project_slug`、`project_name`、`external_workspace_id`、`external_user_id` 和 `external_project_id`。

范围：`deployments:write`。

## 发布
```http
POST /api/v1/deployments/:id/publish
```

身体：
```json
{
  "source_sandbox_id": "sbx_...",
  "kind": "auto",
  "environment": "production",
  "output_path": "/workspace",
  "build_command": "npm run build",
  "run_command": "npm start",
  "port": 3000,
  "health_check_path": "/_health",
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "project_id": "660e8400-e29b-41d4-a716-446655440001",
  "external_workspace_id": "acct_123"
}
```

服务器端行为记录在 [Publishing](/docs/docker-host/publishing) 中。

回应：
```json
{
  "data": {
    "deployment": {
      "id": "dep_...",
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "project_id": "660e8400-e29b-41d4-a716-446655440001",
      "active_version_id": "ver_...",
      "state": "running",
      "public_url": "https://team-portal.example-org.miosa.app"
    },
    "version": {
      "id": "ver_...",
      "kind": "static",
      "state": "ready",
      "artifact_uri": "s3://...",
      "artifact_sha256": "...",
      "source_sha256": "..."
    },
    "release": {
      "id": "rel_...",
      "state": "ready"
    },
    "promoted": true
  }
}
```

## 列表
```http
GET /api/v1/deployments
GET /api/v1/deployments?workspace_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/deployments?project_id=660e8400-e29b-41d4-a716-446655440001
GET /api/v1/deployments?external_workspace_id=acct_123
GET /api/v1/deployments?external_user_id=user-456
GET /api/v1/deployments?state=running
```

所有过滤器都是组织范围的服务器端。请参阅[所有权和归属](/docs/platform/attribution)。

## 获取
```http
GET /api/v1/deployments/:id
```

返回完整的部署行，包括 `workspace_id`、`project_id`、`active_version_id`、属性和元数据。

## 更新
```http
PATCH /api/v1/deployments/:id
```

主体：可变字段的任何子集（`name`、`auto_deploy`、`metadata`、`external_*`）。

## 回滚
```http
POST /api/v1/deployments/:id/rollback
```

身体：
```json
{
  "version_id": "ver_..."
}
```

请参阅[回滚](/docs/docker-host/rollback)。

## 删除
```http
DELETE /api/v1/deployments/:id
```

拆除运行时实例、删除路由、将部署标记为已删除。版本和发布将根据保留策略进行保留。

## 另请参阅

- [Versions](/docs/api-reference/versions) - 版本子资源
- [发布](/docs/api-reference/releases) - 构建工件参考
- [自定义域](/docs/api-reference/custom-domains) - 域子资源
- [部署/概述](/docs/docker-host/overview) - 概念模型
