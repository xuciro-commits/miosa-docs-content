---
slug: "data/postgres"
title: "Postgres"
description: "通过生命周期控制、备份、恢复和 DATABASE_URL 注入来管理每个项目的 Postgres。"
section: "Databases & storage"
order: 4001
---

> **提示：可用**
>
> 托管 Postgres 支持专用实例、`DATABASE_URL` 注入、持久存储、生命周期操作、按需备份以及通过 CLI 和 API 进行恢复。

## 规定

当使用 `data: { postgres: true }` 创建项目时，MIOSA 在托管 Postgres 集群中配置逻辑数据库，生成凭据，并将其作为环境变量注入。

```mermaid
graph LR
    PC[Project create] -->|data.postgres=true| MP[MIOSA provisioner]
    MP --> PG[(Managed Postgres cluster)]
    PG -->|credentials generated| ENV[DATABASE_URL env var]
    ENV --> SB[Sandbox]
    ENV --> RI[Runtime Instance]
```
```typescript
const project = await miosa.projects.create({
  name: "Team Portal",
  data: { postgres: true },
})

// Sandboxes and runtime instances under this project boot with:
// DATABASE_URL = postgresql://user:pass@host:port/proj_xxx
```

您还可以将 Postgres 添加到现有项目：
```typescript
await miosa.projects.dataServices.create(projectId, {
  type: "postgres",
  size: "small",  // small | medium | large
})
```

## 按照您的方式创建：API、CLI 或 SDK

您不需要 SDK。可以通过三种方式创建和使用相同的托管数据库。所有这些都使用相同的 API。

**原始API（curl）：**
```bash
# 1. Create a Postgres database
curl -X POST https://api.miosa.ai/api/v1/databases \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "my-db", "engine": "postgresql"}'
# → { "id": "db_...", "state": "provisioning", ... }   (reaches "running" in ~30-60s)

# 2. Get the connection string once it's running
curl https://api.miosa.ai/api/v1/databases/DB_ID/credentials \
  -H "Authorization: Bearer $MIOSA_API_KEY"
# → { "data": { "recommended_url": "postgresql://user:pass@host:port/miosa_db", ... } }

# 3. Connect with anything that speaks Postgres
psql "postgresql://user:pass@host:port/miosa_db"
```
其他引擎：传递`"engine": "mysql"`、`"redis"`或`"qdrant"`。

**命令行界面：**
```bash
miosa api POST /databases -d '{"name": "my-db", "engine": "postgresql"}'  # create
miosa db list                     # see them + their state
miosa db connection <id>                    # open a shell to it
miosa db logs <id>                       # tail logs
miosa db backup <id>                     # snapshot
miosa api POST /databases/<id>/restore \
  -d '{"backup_id": "<backup-id>"}'          # restore
```

**SDK** （下面是 Python 示例；TypeScript/Go/Elixir SDK 具有相同的方法）。

## 连接管理

使用有界应用程序连接池并根据数据库计划调整其大小。
将凭证端点返回的连接 URL 视为权威的。
不要假定一个单独的会话或事务池，除非返回的数据库配置明确声明了一个。

## 您可以从哪里连接

连接 URL 指向私有 MIOSA 地址。
可以从组织中的沙箱、计算机和部署访问它。
无法从公共互联网访问它，因此笔记本电脑上的 `psql` 无法直接使用它。
要从您自己的计算机上使用数据库，请在沙箱内运行 `psql`，或通过其中打开一条隧道。

## 模式管理由您负责

MIOSA 不会为您运行迁移。推荐的工作流程：

1. 使用框架的迁移工具（Prisma、Drizzle、Ecto、Alembic 等）。
2. 将迁移作为构建或启动的一部分运行，或者作为单独的一次性运行：

```typescript
await sandbox.exec({ cmd: "npx", args: ["prisma", "migrate", "deploy"] })
```

3. 使迁移向后兼容（扩展→迁移→收缩）。这可以保持回滚工作 - 请参阅[回滚](/docs/docker-host/rollback)。

## 所有者权限

连接 URL 中的用户是数据库的所有者角色。
默认情况下，它拥有数据库及其创建的所有内容，仅此而已：它无法创建角色或数据库，并且行级安全性适用于它。

有些迁移需要更多。
为 Supabase 或 Railway 编写的迁移通常会创建自己的角色，授予名为 `postgres` 的角色，运行 `ALTER DEFAULT PRIVILEGES FOR ROLE postgres`，或依赖所有者读取行级安全性。
对于这些，请打开 **提升的** 所有者：

| |标准（默认）|高架|
|---|---|---|
|拥有数据库及其对象 |是的 |是的 |
| `CREATEROLE`（创建并授予您自己的角色）|没有 |是的 |
| `CREATEDB` |没有 |是的 |
| `BYPASSRLS`（读取过去的行级安全性）|没有 |是的 |
|名为 `postgres` 的 `NOLOGIN` 角色的成员 |没有 |是的 |
|超级用户|没有 |没有 |

提升的所有者永远不是超级用户。
`COPY ... TO PROGRAM`、不受信任的语言，例如 `plpython3u`、`pg_read_server_files`、`ALTER SYSTEM` 以及 [允许列表 ](/docs/data/postgres#extensions) 之外的扩展程序将被拒绝。
更改 `SUPERUSER` 属性（例如 `ALTER ROLE app NOSUPERUSER` 的迁移仍然会失败：删除该子句，因为您创建的角色无论如何都不是超级用户。

它在 PostgreSQL 16 或更高版本的付费计划（开发人员及更高版本）上可用，并且除非您为数据库打开它，否则它是关闭的。

**创建数据库时打开它：**

```bash
miosa db create app-db --version 16 --owner-privileges elevated
```

```bash
curl -X POST https://api.miosa.ai/api/v1/databases \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "app-db", "engine": "postgresql", "owner_privileges": true}'
```

**或更改现有数据库：**

```bash
miosa db update app-db --owner-privileges elevated
miosa db update app-db --owner-privileges standard
```

```bash
curl -X PATCH https://api.miosa.ai/api/v1/databases/DB_ID \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"owner_privileges": true}'
```

在正在运行的数据库上，更改会在请求返回之前应用。
在已停止的数据库上，它将保存并在数据库下次启动时应用。
`miosa db get app-db` 和 API 的 `owner_privileges` 字段显示当前设置。

返回标准会再次删除属性和 `postgres` 成员资格。
它不会删除在所有者提升时创建的角色或数据库：如果您不再需要它们，请自行删除它们。
您的组织所做的每项更改都会在其审核日志中记录为 `database.owner_privileges.enabled` 或 `database.owner_privileges.disabled`，以及进行更改的成员或 API 密钥。> **注意：谁可以使用它**
>
> 拥有所有者连接 URL 的任何人都可以获得这些权限：他们可以创建可以登录的角色，并读取行级安全性将隐藏的行。
> 如果您的应用程序不需要，则为您的应用程序提供一个单独的、权限较低的角色。

## 扩展

这些扩展可在数据库中使用：`vector`、`pg_trgm`、`pgcrypto`、`uuid-ossp`、`citext`、`hstore`、`btree_gin`、`btree_gist` 和 `pg_stat_statements`。创建数据库时询问您想要的：

```bash
miosa db create app-db --extensions vector,pg_trgm
```

其他任何内容（例如 `adminpack` 或 `file_fdw`）都会被拒绝。

### pg_stat_statements 默认开启

每个 **新** Postgres 数据库都打开了 `pg_stat_statements`，因此控制台的顶级查询视图和您自己的 `SELECT ... FROM pg_stat_statements` 从第一分钟就开始工作。它被添加到您要求的列表中，并且 `pg_stat_statements.track` 是 Postgres 默认值 (`top`)。

要创建没有它的数据库，请在创建它时发送 `"pg_stat_statements": false`：

```bash
miosa api POST /databases -d '{"name":"app-db","engine":"postgres","pg_stat_statements":false}'
```

已存在的数据库不会更改。要为其中之一打开它，请在该数据库上启用扩展；下次重启时生效。

## 备份

MIOSA 支持存储在数据库运行时之外的计划备份和按需备份。
保留取决于数据库计划。

创建按需备份：

```bash
miosa db backup <database-id> --json
```

从选定的备份恢复：

```bash
miosa api POST /databases/<database-id>/restore \
  -d '{"backup_id": "<backup-id>"}'
```

恢复替换数据库内容。
在确认操作之前，请检查目标数据库和备份ID。

## 生命周期和恢复

创建创建配置；配置准备就绪后即可运行；停止时运行变为停止，启动时返回运行。

重新启动或维护时，运行将转为重新启动；重新启动后将进入已验证运行状态。

恢复失败时重新启动会出错。

|状态 |意义|
|---|---|
| `provisioning` | MIOSA 正在创建存储、凭证和数据库运行时。 |
| `running` |数据库接受连接。 |
| `stopped` |数据库被有意停止。持久数据仍然附加。 |
| `restarting` | MIOSA 正在维护或重新启动请求后恢复所需运行的数据库。 |
| `error` |恢复未完成，需要操作员采取行动。 |

在主机维护期间，MIOSA 会保留数据库 ID、凭据、持久卷、绑定和所需状态。
在其主机恢复活动服务之前，正在运行的数据库将重新启动并进行验证。
故意停止的数据库仍保持停止状态。

## 使用相同数据库的沙盒开发

当您在具有 Postgres 的项目下创建沙箱时，沙箱也会获得 `DATABASE_URL` - 指向同一托管数据库。两种选择：

1. **共享开发数据库。** 沙箱和生产都指向生产数据库。快速迭代，开发工作影响产品数据的风险。
2. **每个环境数据库。** 生产获得一个数据库，登台/开发获得另一个数据库。通过[环境](/docs/docker-host/environments)配置。

默认值为 (2) 每个环境隔离着陆一次。在此之前，沙箱共享生产数据库；使用模式命名空间或小心破坏性查询。

## 从沙箱连接
```typescript
await sandbox.exec({
  cmd: "psql",
  args: [process.env.DATABASE_URL, "-c", "SELECT 1"],
})
```

沙箱的 `DATABASE_URL` 与运行时在生产中看到的形状相同。无论沙箱中有效的代码，在部署中都有效。

## 限制

|等级 |存储|连接（PgBouncer）|内存|
|---|---|---|---|
|小| 1 GB | 100 | 512 MB|
|中等| 10GB| 200 | 2GB|
|大| 100GB| 500 | 8GB|

存储、连接和内存限制取决于数据库计划。
检查数据库响应和指标，而不是假设存储自动增长。

## 成本

Postgres 的定价分为两部分：

- **计算** - 数据库运行时每小时（默认情况下生产始终开启；对于沙盒开发数据库可以暂停）。
- **存储** - GB 月数据加 WAL。

请参阅[使用情况和计费](/docs/platform/usage-and-billing)。每个数据库的计量带有[归因](/docs/platform/attribution)，因此平台可以按最终客户收费。

## 带上你自己的 Postgres？如果您想使用外部 Postgres，只需将 `DATABASE_URL` 设置为部署上的环境变量并跳过托管产品：
```typescript
await miosa.deployments.env.set(deploymentId, {
  environment: "production",
  vars: { DATABASE_URL: "postgresql://...your-external-db..." },
})
```

MIOSA 的运行时会注入您设置的任何内容。托管的 Postgres 很方便；支持自带。

## Python SDK

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Provision a managed database
db = miosa.databases.create(
    name="team-portal-db",
    engine="postgres",
    size="small",
)
print(db["id"])  # db_...

# Get connection credentials
creds = miosa.databases.credentials(db["id"])
print(creds["url"])  # postgresql://user:pass@host:port/dbname

# Lifecycle operations
miosa.databases.stop(db["id"])
miosa.databases.start(db["id"])
miosa.databases.restart(db["id"])

# Stream logs (returns a generator of SSE events)
for event in miosa.databases.stream_logs(db["id"]):
    print(event)

# Teardown
miosa.databases.delete(db["id"])
```

## 另请参阅

- [概述](/docs/data/overview)：更广泛的数据平面
- [MySQL](/docs/data/mysql)：如果您需要 MySQL 兼容性
- [环境](/docs/docker-host/environments)：每个环境的凭据
- [Rollback](/docs/docker-host/rollback)：为什么向后兼容的迁移很重要
