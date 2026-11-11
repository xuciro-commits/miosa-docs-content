---
slug: "concepts/data-and-bindings"
title: "数据和绑定"
description: "托管 Postgres、Redis、对象存储、卷和身份验证 - 部署附加到的持久资源，以及绑定如何注入凭据。"
section: "Optional"
order: 17011
---

沙箱和运行时实例是可替换的。
数据服务是具有独立生命周期的持久资源。
替换沙盒、发布版本或回滚版本不会删除部署的数据库或对象存储。

|服务 |它是什么 |
|--------|------------|
|托管 Postgres |完全托管的 PostgreSQL 数据库。架构迁移由您来运行； MIOSA 提供连接字符串。 |
|托管 Redis |用于缓存、队列、发布/订阅的持久 Redis 集群。 |
|对象存储 | S3 兼容存储桶。对于上传、资产、生成的文件很有用。 |
|卷 |块存储安装到运行时实例中。重新启动后仍然存在。 |
|授权 |基于 JWT 的注册/登录即服务。一个环境变量为您的项目提供了完整的身份验证系统。 |

绑定将沙箱或部署连接到这些持久资源。
MIOSA 解析绑定并在运行时注入适当的凭据。

```bash
DATABASE_URL=postgresql://user:pass@host/dbname
REDIS_URL=redis://host:6379
STORAGE_BUCKET=miosa-ws123-uploads
AUTH_URL=https://auth.miosa.app/proj_abc
AUTH_JWT_SECRET          # 由 MIOSA 注入，请勿填写
```

销毁沙箱并不会破坏其数据库。
发布版本不会自动使不兼容的架构变得安全。
使用向后兼容的扩展、迁移和收缩更改，以便活动版本和以前的版本都可以在升级或回滚期间运行。

> **注：阶段状态**
>
> 托管 Postgres 和 Auth 是平台第 6-8 阶段部署的一部分。文档界面现已发布，以便您可以规划集成；一些端点仍在生产中连接。

```bash
miosa api POST /databases -d '{"name":"app-production","engine":"postgresql"}'
```

[数据概览→](/docs/data/overview)

## 下一步

- [发布管道](/docs/concepts/publish-pipeline)：发布如何附加到这些资源。
- [数据概述](/docs/data/overview)：完整的数据服务参考。
- [核心概念](/docs/concepts)：一屏概述。
