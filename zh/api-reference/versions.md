---
slug: "api-reference/versions"
title: "API 参考/版本"
description: "不可变的部署版本子资源。列出、获取、推广、存档。"
section: "API reference"
order: 12040
---

**部署版本**是一次发布的不可变记录。请参阅[版本](/docs/docker-host/versions)了解概念模型。

## 端点
```http
GET  /api/v1/deployments/:id/versions
GET  /api/v1/deployments/:id/versions/:version_id
POST /api/v1/deployments/:id/versions/:version_id/promote
```

## 列表
```http
GET /api/v1/deployments/:id/versions
GET /api/v1/deployments/:id/versions?state=ready
GET /api/v1/deployments/:id/versions?workspace_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/deployments/:id/versions?external_workspace_id=acct_123
GET /api/v1/deployments/:id/versions?limit=50
```

回应：
```json
{
  "data": [
    {
      "id": "ver_...",
      "deployment_id": "dep_...",
      "version_number": 17,
      "kind": "static",
      "state": "ready",
      "artifact_sha256": "a5e6f0c1...",
      "source_sha256": "f1a3b8c2...",
      "promoted_at": "2026-05-14T18:20:00Z",
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "project_id": "660e8400-e29b-41d4-a716-446655440001",
      "external_workspace_id": "acct_123",
      "external_user_id": "user-456",
      "created_at": "2026-05-14T18:19:48Z"
    }
  ],
  "total": 17
}
```

## 获取
```http
GET /api/v1/deployments/:id/versions/:version_id
```

返回完整版本行，包括 `artifact_manifest`、`runtime_image`、`runtime_command`、`runtime_port`、`health_check_path`、`build_log_uri`。

## 推广

使特定的就绪版本成为部署的活动版本。
```http
POST /api/v1/deployments/:id/versions/:version_id/promote
```

正文：空，或可选 `{ "environment": "production" }`。

范围：`deployments:write`。为了重试的安全性，需要 `Idempotency-Key`。

促销与发布不同：
- **发布** 从沙箱源创建新版本。
- **升级** 将现有就绪版本点为环境中的活动版本。

使用提升进行金丝雀/分阶段部署：首先发布到非生产环境，进行验证，然后提升到生产环境。

## 州

|状态 |意义|
|---|---|
| `created` |行存在，构建未开始 |
| `building` |建设进行中 |
| `ready` |构建成功，工件已上传，可推广 |
| `failed` |构建或运行状况检查失败 |
| `archived` |晋升出；仍可启动以进行回滚|

状态转换是单向的。一旦 `ready` 或 `failed`，版本就会保持这种状态。

## 权限

|行动|范围 |
|---|---|
|列出、获取 | `deployments:read` |
|推广| `deployments:write` |

## 另请参阅

- [部署](/docs/api-reference/deployments) - 父资源
- [发布](/docs/api-reference/releases) - 工件参考
- [版本](/docs/docker-host/versions) - 概念模型
- [Rollback](/docs/docker-host/rollback) - 在内部使用推广
