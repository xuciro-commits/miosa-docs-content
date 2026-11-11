---
slug: "data/mysql"
title: "MySQL"
description: "通过持久存储和 DATABASE_URL 注入来管理每个项目的 MySQL。"
section: "Databases & storage"
order: 4002
---

> **提示：可用**
>
> 托管 MySQL 适用于需要 MySQL 兼容架构的项目，
> 框架，或导入。 MIOSA 配置实例、存储凭证并
> 将 `DATABASE_URL` 注入附加的沙箱和生产运行时。

## 何时使用 MySQL

当应用程序已经需要 MySQL 语义时，使用托管 MySQL：

- Laravel、WordPress 或 MySQL 优先的框架
- 不应转换为 Postgres 的现有客户架构
- 依赖于 MySQL 特定行为的 SQL 工作负载
- 随 MySQL 迁移一起提供的 SaaS 模板

当你需要更丰富的关系功能时，使用[Postgres](/docs/data/postgres)，
扩展或 Postgres 原生工具。

## 运行时合约

应用程序不需要知道数据库在哪里运行。读取注入的内容
启动时的连接字符串：

```bash
DATABASE_URL=mysql://user:password@host:3306/database
```

MIOSA 将相同的变量注入：

- 用于开发和智能体工作的沙箱
- 用于生产部署的运行时实例
- 属于该项目的后台工作者

生命周期、备份、恢复、凭证、日志和指标使用通用的[数据库 API](/docs/api-reference/databases)。
不要从主机名或项目 slug 推断这些值。

## 配置模式

```typescript
const project = await miosa.projects.create({
  name: "Customer Portal",
  data: { mysql: true },
});
```

对于应用程序构建器和白标平台，为每个客户项目创建 MySQL
当生成的应用程序需要它时。将架构迁移保留在应用程序中
存储库或沙箱工作区，以便部署保持可重复性。

## 另请参阅

- [数据概述](/docs/data/overview)：为每个工作负载选择正确的数据服务。
- [Postgres](/docs/data/postgres)：Postgres 本机应用程序的托管关系数据库。
- [环境](/docs/docker-host/environments)：单独的生产、登台和开发凭据。
