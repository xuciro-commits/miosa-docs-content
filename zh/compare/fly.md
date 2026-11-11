---
slug: "compare/fly"
title: "MIOSA 与 Fly.io"
description: "使用截至 2026 年 10 月 23 日的 Fly.io 公共文档，对 MIOSA 和 Fly.io 在运行应用程序和机器方面进行了事实比较。"
section: "Optional"
order: 17016
---

比较

Fly.io 将应用程序作为靠近用户的机器来运行。 MIOSA 在一个平台上运行服务、计算机和沙箱。此页面比较每个文档的内容。

每个竞争对手的声明均来自 Fly.io 自己的公开文档（[fly.io/docs](https://fly.io/docs)，截至 2026 年 10 月 23 日）。 MIOSA 声明链接到记录这些声明的页面。 MIOSA 没有对应的地方，该行就是这样说的。

## 工作负载

|能力|MIOSA | Fly.io |
|---|---|---|
|来自图像或存储库的长时间运行的应用程序 |是的，服务运行模式 `server`、`worker`、`cron`、`static` |是的，机器上的应用程序 |
|直接控制虚拟机|是的，具有完整 Linux 桌面的计算机 |是的，机器|
|短暂的代码执行 |是的，沙箱 |不是记录的模型；机器默认是长寿的 |
|配置为代码 | `miosa.json` / `miosa.app.yml` | `fly.toml` |
|多个地区 | `us-east`、`us-west`、`us-mia` |更广泛的已发布地区列表 |

来源：[服务](/docs/services/overview)、[计算机](/docs/computers/overview)、[沙箱](/docs/develop/sandboxes)和Fly.io的[机器](https://fly.io/docs/machines/)和[fly.toml参考](https://fly.io/docs/reference/configuration/)。

## 网络和数据

|能力|MIOSA | Fly.io |
|---|---|---|
|项目服务之间的专用网络 |今天是同一位主持人；更广泛的专用网络并不普遍可用|记录了跨区域的专用网络 |
|每项服务的公共地址 |是的，`<label>.<org>.miosa.app`，加上自定义域 |是的，`*.fly.dev` 加上证书 |
|托管 Postgres、MySQL、Redis、MongoDB |是的 |托管 Postgres 已记录；其他引擎由您运行 |
|具有白名单的公共数据库端点 |是的，仅限 TLS |通过您自己的智能体或应用程序记录连接 |

来源：[自定义域](/docs/docker-host/domains)、[数据库](/docs/data/overview)、[公共端点](/docs/data/public-endpoint)。

## MIOSA 没有什么

- **Fly.io 的区域覆盖范围。** 三个区域，如上所述；没有其他要求。
- **服务之间的跨主机专用网络。** 同主机专用网络现在可以使用。

## 何时选择每个

- **如果您需要其区域覆盖范围或者您已经在其周围运行网络，请选择 Fly.io**。
- **如果您的产品还需要沙箱和智能体工作负载、您自己的客户的控制台或机器、服务和数据的一个平台，请选择 MIOSA**。

> **注意：决定前请检查**
>
> 两种产品均发生变化。上面的链接是本页面于 2026 年 10 月 23 日撰写的内容。
