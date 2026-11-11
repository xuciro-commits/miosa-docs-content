---
slug: "data/overview"
title: "数据"
description: "具有明确生命周期、凭证和恢复操作的托管数据库、对象存储、身份验证和卷。"
section: "Databases & storage"
order: 4000
---

MIOSA 数据服务使您的沙箱和生产运行时实例可以访问持久存储，而无需设置任何外部帐户。当您将数据服务附加到项目时，MIOSA 会配置资源并在启动时注入其凭据 - 您的代码读取 `process.env.DATABASE_URL` 并连接。这就是整个集成。

现在就做

创建您的第一个数据库

具有可作为 DATABASE_URL 注入的凭据的托管 Postgres 实例。

- [创建数据库](https://miosa.ai/databases/new)：（在新选项卡中打开 MIOSA 控制台）

miosa db 创建 postgres --name my-db --wait

下一页 [托管 Postgres](/docs/data/postgres)

## 阶段状态

> **注：当前服务状态**
>
> 数据服务逐步交付。该表反映了经过验证的平台状态：
>
> |服务 |状态 |
> |---|---|
> |对象存储 | **可用** - 存储桶、对象、预签名 URL、配额、每个租户 |
> |计划任务 | **可用** - CLI、MCP、SDK 和后端全部有线 |
> |托管 Postgres | **普遍可用** - 专用实例、凭证注入、生命周期操作、备份和恢复 |
> |托管 MySQL | **普遍可用** - 专用实例、凭证注入和生命周期操作 |
> |托管 Redis | **普遍可用** - 每个项目实例和 `REDIS_URL` 注入 |
> |托管 MongoDB | **普遍可用** - 每个项目的文档数据库和 `MONGODB_URI` 注入 |
> |托管 Qdrant（矢量）| **普遍可用** - 每个项目向量数据库和 `QDRANT_URL` 注入 |
> |身份验证（每个项目最终用户身份验证）| **普遍可用** - 选择注册、登录、JWT、密码重置和电子邮件确认 |
> |卷（持久块存储）| **普遍可用** - 创建、附加、分离、删除和计量主机本地卷 |
>
> 每页标注指示任何正在进行的边缘情况。

## 心理模型

每个服务扮演不同的角色。为您的数据形状选择正确的一个：

|服务 |何时需要 |持久性 |
|---|---|---|
| **Postgres** |关系数据、连接、事务、迁移 |持久 - 经受住部署和沙箱销毁 |
| **MySQL** |兼容 MySQL 的应用程序、现有 MySQL 架构 |持久 - 经受住部署和沙箱销毁 |
| **Redis** |缓存、会话、发布/订阅、速率限制 |持久（AOF/RDB），但按缓存类对待 |
| **MongoDB** |文档数据、灵活的模式 |持久 - 经受住部署和沙箱销毁 |
| **Qdrant** |向量搜索、语义相似度、嵌入 |持久 - 集合数据在虚拟机之间保留 |
| **Storage** |文件上传、生成的资产、二进制 blob |持久 - 内容寻址的对象存储 |
| **Auth** |按项目的最终用户注册、登录和 JWT 签发 |持久 - 用户记录比任何虚拟机都长寿 |
| **Volumes** |不适合放在 Postgres 中的大型或高速文件系统状态 |同一时间只附加到一个运行时实例 |

## 数据在您的虚拟机之外仍然存在

沙箱默认是持久的，可以暂停、恢复、快照或销毁。
部署运行时实例是可替换的。
托管数据具有独立的生命周期，并且在更换任一计算表面时不会被删除。

销毁沙箱会删除虚拟机 - 连接的 Postgres 数据库不会受到影响。发布新版本不会迁移您的架构。回滚部署不会回滚您的数据库。数据服务的管理独立于连接到它们的计算。

这是故意的。这也是生产在部署期间保持可用的原因：新版本连接到旧版本使用的同一数据库。

## 凭证如何注入

MIOSA 可以将不同的数据库凭证绑定到不同的部署环境：
```text

production runtime  →  DATABASE_URL = postgres://prod-db...
staging runtime     →  DATABASE_URL = postgres://staging-db...
```

您提供一个环境变量名称，MIOSA 会解析该环境的绑定。
必须显式配置环境隔离。
在附加单独的开发和生产数据库之前，沙箱可以共享生产数据库。
请参阅[环境](/docs/docker-host/environments)。

## 简单示例

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

# Create a managed Postgres database
db = miosa.databases.create(
    engine="postgres",
    engine_version="15",
    name="my-app-db",
)

print(db["id"])          # db_...
print(db["status"])      # provisioning | running

# Fetch connection credentials once ready
creds = miosa.databases.credentials(db["id"])
print(creds["url"])      # postgresql://user:pass@host:5432/dbname
```

**打字稿**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

// Create a managed Postgres database
const db = await miosa.databases.create({
  engine: "postgres",
  engineVersion: "15",
  name: "my-app-db",
})

console.log(db.id)       // db_...
console.log(db.status)   // provisioning | running

// Fetch connection credentials once ready
const creds = await miosa.databases.credentials(db.id)
console.log(creds.url)   // postgresql://user:pass@host:5432/dbname
```

**curl**

```bash
# Create a managed Postgres database
curl -X POST https://api.miosa.ai/api/v1/databases \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"engine":"postgres","engine_version":"15","name":"my-app-db"}'

# Get credentials
curl https://api.miosa.ai/api/v1/databases/$DATABASE_ID/credentials \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## 数据服务如何附加
```mermaid
graph TB
    DEP[Deployment / Sandbox]
    DEP -->|DATABASE_URL injected| PG[(Postgres)]
    DEP -->|DATABASE_URL injected| MY[(MySQL)]
    DEP -->|REDIS_URL injected| RD[(Redis)]
    DEP -->|QDRANT_URL injected| QD[(Qdrant)]
    DEP -->|STORAGE_BUCKET + STORAGE_ENDPOINT + keys injected| ST[(Object Storage)]
    DEP -->|AUTH_URL + AUTH_JWT_SECRET injected| AU[Auth Service]
    DEP -->|mount path injected| VL[Volume]
```

## 它如何组合在一起
```mermaid
flowchart LR
  P[Project] --> S["Sandbox<br/>persistent-by-default development compute"]
  P --> D["Data services (durable)<br/>Postgres, MySQL, Redis, MongoDB, Qdrant, Storage, Auth (opt-in), Volumes"]
  D -->|"connection env vars injected at boot"| S
```您的应用程序代码读取标准环境变量并打开正常的客户端连接。
应用程序仍然负责架构迁移、向后兼容性和破坏性操作的安全处理。

## 托管数据库生命周期

托管数据库公开显式生命周期和恢复操作：

创建创建配置；配置准备就绪后即可运行；停止时运行变为停止，启动时返回运行。

重新启动或维护时，运行将转为重新启动；重新启动后将进入已验证运行状态。

恢复失败时重新启动会出错。

API 支持启动、停止、重新启动、唤醒、调整大小、分支、副本创建、备份、恢复、移动、凭据、日志和指标。
当前的 CLI 公开了常见的创建、检查、生命周期、连接、备份、恢复、日志和指标操作。
请参阅 [Databases API](/docs/api-reference/databases) 了解确切的端点。

## 相关

- [Postgres](/docs/data/postgres)：每个项目的托管 Postgres，包括生命周期操作、备份、恢复和 DATABASE_URL 注入。
- [MySQL](/docs/data/mysql)：每个项目的托管 MySQL。 DATABASE_URL 注入、专用实例、数据在虚拟机销毁后仍然存在。
- [Redis](/docs/data/redis)：每个项目托管的 Redis。缓存、会话存储和发布/订阅。 REDIS_URL 在启动时注入。
- [MongoDB](/docs/data/mongodb)：每个项目的托管 MongoDB。文档数据。 MONGODB_URI 在启动时注入。
- [Qdrant (Vector)](/docs/data/qdrant)：每个项目的矢量数据库。语义搜索和嵌入存储。 QDRANT_URL 在启动时注入。
- [存储](/docs/data/storage)：每个项目的对象存储桶。签名 URL 上传和下载，浏览器中无需凭据。
- [Auth](/docs/data/auth)：选择加入每个项目的最终用户身份验证。注册、登录、JWT。 `AUTH_URL` + `AUTH_JWT_SECRET` 在启动时注入。
- [交易电子邮件](/docs/data/email)：重新发送验证电子邮件，用于密码重置和电子邮件确认。操作员级别设置，而不是每个项目。
- [Volumes](/docs/data/volumes)：附加到运行时实例的持久块存储。数据在虚拟机销毁后仍然存在并重新连接。
- [分形计算](/docs/data/fractal)：用于客户端范围的结构化数据、大容量计算、协调和私有数据平面部署的企业数据合作伙伴模式。

## 后续步骤

- [Connect Postgres](/docs/data/postgres)：凭证、DATABASE_URL 注入、备份和恢复。
- [添加对象存储](/docs/data/storage)：在浏览器中无需密钥的存储桶和签名 URL 上传。
- [附加卷](/docs/data/volumes)：在虚拟机销毁后仍然存在的持久块存储。
