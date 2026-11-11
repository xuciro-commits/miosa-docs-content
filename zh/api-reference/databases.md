---
slug: "api-reference/databases"
title: "数据库API"
description: "配置和操作托管数据库、备份、恢复、分支、副本、凭证和放置。"
section: "API reference"
order: 12042
---

托管数据库具有独立于沙箱和部署运行时实例的生命周期。
删除或替换计算不会隐式删除附加的数据库。

基本路径：`/api/v1/databases`

`managed-databases` 别名不可用：使用 `/api/v1/databases`。

## 核心生命周期

|方法|路径|目的|
|---|---|---|
| `GET` | `/databases` |列出当前租户和工作区范围可见的数据库 |
| `POST` | `/databases` |创建托管数据库 |
| `GET` | `/databases/{id}` |检查一个数据库 |
| `PATCH` | `/databases/{id}` |重命名它、更改其磁盘大小或更改其所有者权限 |
| `DELETE` | `/databases/{id}` |永久删除一个数据库 |
| `POST` | `/databases/{id}/start` |启动已停止或可恢复的数据库 |
| `POST` | `/databases/{id}/stop` |停止正在运行的数据库，同时保留其卷 |
| `POST` | `/databases/{id}/restart` |重新启动数据库运行时 |
| `POST` | `/databases/{id}/wake` |唤醒可恢复的数据库|
| `POST` | `/databases/{id}/resize-compute` |更改其计算分配 |
| `POST` | `/databases/{id}/move` |通过自选展示位置工作流程移动数据库 |

## 数据拓扑和恢复

|方法|路径|目的|
|---|---|---|
| `POST` | `/databases/{id}/branch` |创建独立的数据库分支 |
| `POST` | `/databases/{id}/replicas` |创建副本 |
| `POST` | `/databases/{id}/backups` |创建按需备份 |
| `GET` | `/databases/{id}/backups` |列出备份 |
| `GET` | `/databases/{id}/backups/{backup_id}` |检查一个备份 |
| `POST` | `/databases/{id}/restore` |从选定的备份恢复 |
| `GET` | `/databases/{id}/restores` |列出恢复操作 |
| `GET` | `/databases/{id}/credentials` |返回当前连接合约 |

日志和指标使用常见的可观察性端点：

```http
GET /api/v1/databases/{id}/logs
GET /api/v1/databases/{id}/logs/stream
GET /api/v1/databases/{id}/metrics
```

## 创建

```bash
curl -X POST https://api.miosa.ai/api/v1/databases \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Idempotency-Key: $IDEMPOTENCY_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"app-production","engine":"postgresql"}'
```

创建是异步的。
等待返回的数据库到达`running`后再进行连接。

```bash
miosa api POST /databases -d '{"name":"app-production","engine":"postgresql"}'
```

## 所有者权限

`owner_privileges` （布尔值，默认 `false`）给出 PostgreSQL 16+ 数据库的所有者角色 `CREATEROLE`、`CREATEDB`、`BYPASSRLS` 以及名为 `postgres` 的 `NOLOGIN` 角色的成员资格。
它永远不是超级用户。
将其发送到 `POST /databases` 以使用它创建数据库，或发送到 `PATCH /databases/{id}` 以打开或关闭它。
每个数据库响应都携带当前值。
请参阅 [Postgres: 所有者权限](/docs/data/postgres#owner-privileges) 了解它允许什么以及何时使用它。

```bash
miosa api PATCH /databases/<database-id> -d '{"owner_privileges": true}'
```

拒绝：

- `402 PLAN_LIMIT_EXCEEDED`：开启它需要付费套餐。始终允许将其关闭。
- `422 OWNER_PRIVILEGES_UNSUPPORTED`：数据库不是 PostgreSQL 16 或更高版本。
- `503 OWNER_PRIVILEGES_NOT_APPLIED`：数据库正在运行，其主机无法应用更改。一切都没有改变；重试。

## 凭证

```bash
curl https://api.miosa.ai/api/v1/databases/$DATABASE_ID/credentials \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

使用返回的推荐连接 URL。
不要从资源元数据构建主机名、端口、用户名或数据库名称。

## 备份与恢复

```bash
miosa db backup <database-id> --json
miosa api POST /databases/<database-id>/restore \
  -d '{"backup_id": "<backup-id>", "allow_overwrite": true}'
```

还原将目标数据库内容替换为选定的备份。
批准操作前请确认数据库 ID 和备份 ID。

> **注意：应用程序迁移是单独的**
>
> 数据库恢复不会更改活动应用程序版本。
> 应用程序回滚不会逆转数据库迁移。
> 当版本必须保持回滚安全时，使用向后兼容的扩展、迁移和收缩更改。

## 权限

|行动|所需范围 |
|---|---|
|列出、检查、读取指标或获取凭据 | `databases:read` |
|创建、更新、调整大小、分支、备份、恢复、移动或删除 | `databases:write` |

## 另请参阅

- [数据概述](/docs/data/overview)：选择正确的托管数据产品并了解其生命周期。
- [Postgres](/docs/data/postgres)：安全地连接应用程序并管理架构更改。
- [环境](/docs/docker-host/environments)：将单独的数据库绑定到生产、暂存和开发。
