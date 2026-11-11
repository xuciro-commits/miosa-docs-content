---
slug: "compare/railway"
title: "MIOSA 与 Railway"
description: "使用截至 2026 年 10 月 23 日的 Railway 公共文档，对 MIOSA 和 Railway 运行服务进行事实比较。"
section: "Optional"
order: 17014
---

比较

Railway 和 MIOSA 都能从代码仓库运行服务。本页面比较两者各自的文档，帮助您判断哪一种更契合您的工作方式。

以下每个竞争对手的声明均来自 Railway 自己的公开文档（[docs.railway.com](https://docs.railway.com/)，截至 2026 年 10 月 23 日）。 MIOSA 声明链接到记录这些声明的页面。 MIOSA 没有对应的地方，该行就是这样说的。

## Services

|能力|MIOSA |Railway|
|---|---|---|
|来自存储库的 Web 服务 |是的，有公共地址 |是的 |
|后台 Worker |是的，运行模式 `worker` |是的 |
|定时任务 |是的，运行模式 `cron`，使用五段式时间表 |是的 |
|静态站点|是的，从 git 构建并由静态通道提供服务 |是的 |
|每个拉取请求预览 |是的，每个拉取请求一个，具有相同的上限和 TTL 控制 |是的，PR 环境 |
|配置为代码 | `miosa.json` 或 `miosa.app.yml` 与 Docker 主机共享 | `railway.json` / `railway.toml` |

来源：[服务](/docs/services/overview)、[预览](/docs/services/previews)、[配置文件](/docs/services/config-file)和Railway的[服务](https://docs.railway.com/guides/services)和[环境](https://docs.railway.com/guides/environments)。

## 发布和流量

|能力|MIOSA |Railway|
|---|---|---|
|带回滚的发布 |是的，每个版本都会保留其源代码和设置 |是的 |
|副本 |是的，在网络允许的情况下跨主机分散 |是的 |
|滚动部署 |是的，逐步切换流量并自动回滚 |滚动部署按服务分别记录 |
|日志 |构建和运行时日志，以及通过 HTTPS 或 syslog/TLS 进行的日志消耗 |付费计划中的日志和日志消耗 |

来源：[发布](/docs/services/releases)、[副本](/docs/services/replicas)、[日志](/docs/services/logs)。

## 数据

|能力|MIOSA |Railway|
|---|---|---|
| Postgres、MySQL、Redis |是的，已管理 |是的 |
| MongoDB |是的 |是的 |
|公共连接端点 |是的，按数据库选择加入，仅限 TLS，并有许可名单 | TCP智能体|
|服务上的卷 |是的 |是的 |
|带引用的变量 | `${{Service.NAME}}`、`${{shared.NAME}}`、`${{secret(n)}}` |带有 `${{...}}` | 的参考文献

来源：[数据库](/docs/data/overview)、[公共端点](/docs/data/public-endpoint)、[变量](/docs/services/variables)。

## MIOSA 没有什么

- **服务之间的跨主机专用网络。** 同主机专用网络现在可以使用；更广泛的内容通常不可用，并且本页不会另有声明。
- **Railway的完整区域列表。** MIOSA 在 `us-east`、`us-west` 和 `us-mia` 运行。

## 何时选择每个

- **如果您想要更广泛的区域设置或者您已经在那里定居，请选择Railway**。
- **如果您还想要同一平台上的完整 Linux 计算机和沙箱、一组凭据以及为您自己的客户提供白标界面，请选择 MIOSA**。

> **注意：决定前请检查**
>
> 两种产品都会变化。上面的链接是本页面在 2026-10-23 撰写时所依据的文档；请阅读它们，然后做出决定。

要迁移？ [阅读 Railway 迁移指南](/docs/migrate/railway)，并且 `miosa import railway` 会从 Railway 项目编写可审查的计划，而不复制机密值。
