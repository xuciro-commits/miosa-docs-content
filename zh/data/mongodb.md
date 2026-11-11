---
slug: "data/mongodb"
title: "MongoDB"
description: "管理每个项目的 MongoDB 文档数据。"
section: "Databases & storage"
order: 4005
---

> **提示：可用**
>
> Managed MongoDB 是一个像 Postgres、MySQL 和 Redis 一样的数据库引擎。
> 它在自己的 microVM 中运行官方 MongoDB 7 服务器，并具有持久数据量。

## 你得到什么

具有自动生成的 root 用户和密码的托管 MongoDB 7 实例，注入为 `MONGODB_URI`（也为 `MONGO_URL`）：

```bash
MONGODB_URI=mongodb://user:pass@host:port/miosa_db?authSource=admin
```

标准 MongoDB 驱动程序无需修改即可工作：

```typescript
import { MongoClient } from "mongodb"
const client = new MongoClient(process.env.MONGODB_URI!)
await client.connect()

const users = client.db("miosa_db").collection("users")
await users.insertOne({ name: "Ada", createdAt: new Date() })
```

用户具有 root 角色，因此您可以在实例上创建任意数量的数据库。
`authSource=admin` 是 URL 的一部分，必须保留。

## 规定

```bash
miosa db create --name app-mongo --engine mongodb
```

```typescript
await miosa.databases.create({ name: "app-mongo", engine: "mongodb", size: "small" })
```

您还可以在项目的“添加”菜单中选择 MongoDB，或在更改集中的数据库服务上设置 `managed: mongodb`。
仅主要版本 7 可用。

## 连接变量

|变量|价值|
| ---| ---|
| `MONGODB_URI` |完整连接字符串 |
| `MONGO_URL` |与 `MONGODB_URI` | 相同
| `MONGO_HOST`、`MONGO_PORT` |主机和端口 |
| `MONGO_USER`、`MONGO_PASSWORD` | root用户和密码|
| `MONGO_DATABASE` |默认数据库名称 |

## 备份和恢复

备份是逻辑备份（`mongodump`，gzip 存档）并覆盖实例上除 `admin` 和 `config` 之外的每个数据库。
恢复会替换备份包含的集合，而不会影响其他集合。
它永远不会触及 `admin` 数据库，因此您颁发的凭据继续有效。
MongoDB 备份仅恢复到 MongoDB 数据库中。

## 限制

- MongoDB 没有表浏览器或查询控制台。使用 `mongosh` 或驱动程序。
- 与 MySQL 和 Redis 一样，空闲时不会自动暂停。
- 内存、连接和存储遵循所选的计划大小。

## 另请参阅

- [概述](/docs/data/overview)
- [Redis](/docs/data/redis)
- [Postgres](/docs/data/postgres)
