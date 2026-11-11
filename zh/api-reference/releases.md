---
slug: "api-reference/releases"
title: "发布API"
description: "检查并激活精确的不可变部署工件。"
section: "API reference"
order: 12041
---

**发布**是从构建或沙箱发布生成的不可变工件。
它记录工件摘要、运行时命令、端口、运行状况检查、源属性和用于精确升级的准备状态。

基本路径：`/api/v1/deployments/{deployment_id}/releases`

## 端点

|方法|路径|目的|
|---|---|---|
| `GET` | `/deployments/{deployment_id}/releases` |列出一次部署的版本 |
| `GET` | `/deployments/{deployment_id}/releases/{release_id}` |检查一个确切的版本 |
| `POST` | `/deployments/{deployment_id}/releases/{release_id}/promote` |激活一个确切的版本 |

发布创建候选版本。
仅在请求的版本准备好并且配置的运行时接受它之后，升级才会更改活动的生产工件。

## 列出版本

```bash
curl https://api.miosa.ai/api/v1/deployments/$DEPLOYMENT_ID/releases \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

响应包含 `data`、兼容性 `releases` 数组和 `total`。

```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "deployment_id": "660e8400-e29b-41d4-a716-446655440000",
      "deployment_version_id": "770e8400-e29b-41d4-a716-446655440000",
      "kind": "dynamic",
      "state": "ready",
      "artifact_sha256": "a5e6f0c1...",
      "start_command": "node server.js",
      "port": 3000,
      "health_check_path": "/api/health",
      "ready_at": "2026-07-24T18:19:48Z",
      "created_at": "2026-07-24T18:18:06Z"
    }
  ],
  "total": 1
}
```

## 检查一个版本

```bash
curl \
  https://api.miosa.ai/api/v1/deployments/$DEPLOYMENT_ID/releases/$RELEASE_ID \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

使用此端点在升级之前确认确切的工件摘要、运行时合同、源沙箱、构建和准备状态。

## 推广一个确切的版本

提升是一种突变，需要幂等密钥。

```bash
curl -X POST \
  https://api.miosa.ai/api/v1/deployments/$DEPLOYMENT_ID/releases/$RELEASE_ID/promote \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Idempotency-Key: $IDEMPOTENCY_KEY"
```

响应包含部署以及持久操作记录。
使用相同的幂等键重复相同的请求将返回现有结果，而不是激活释放两次。

> **注意：运行时感知激活**
>
> 升级使用部署的已配置运行时产品。
> 对于 Docker Host，通过工作区 Docker Host 主机激活发布。
> 对于标准 MIOSA Deploy，它是通过标准动态或静态运行时激活的。

## 命令行界面

```bash
miosa deploy releases <deployment-id> --json
miosa deploy versions <deployment-id> --json
miosa deploy promote <deployment-id> <release-id> --json
miosa deploy rollback <deployment-id> --to <release-id> --yes --json
```

## 不变性

发布内容由 `artifact_sha256` 寻址。
更改应用程序内容会创建新版本。
升级永远不会改变现有版本的字节。

回滚选择先前准备好的版本，并通过相同的运行时感知路径激活该确切的工件。

## 权限

|行动|所需范围 |
|---|---|
|列出或检查版本 | `deployments:read` |
|提升或回滚 | `deployments:write` |

## 另请参阅

- [发布](/docs/docker-host/publishing)：从沙箱源创建候选版本。
- [版本](/docs/api-reference/versions)：检查逻辑部署版本及其构建状态。
- [回滚](/docs/docker-host/rollback)：恢复之前正常的版本。
