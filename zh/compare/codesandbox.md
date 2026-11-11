---
slug: "compare/codesandbox"
title: "MIOSA 与 CodeSandbox"
description: "使用截至 2026 年 10 月的独立 ComputeSDK 基准测试和 CodeSandbox 的公共文档对 MIOSA 和 CodeSandbox 进行事实比较。"
section: "Optional"
order: 17027
---

比较

CodeSandbox 和 MIOSA 都在托管云环境中运行代码。本页在独立的 ComputeSDK 基准测试和 MIOSA 功能文档上对它们进行了比较。

## CodeSandbox 是什么

CodeSandbox 提供即时云开发环境和沙箱，其公共文档（[codesandbox.io](https://codesandbox.io/)，截至 2026 年 10 月）对此进行了描述。

## 测量性能

独立的 ComputeSDK 基准测试针对每个提供商运行相同的工作负载。这是这里唯一可以比较的数字。

|公制|MIOSA |代码沙盒 |截至 |
|---|---|---|---|
|突发启动、中位交互时间 | **57.09 毫秒（第 1 条，共 32 条）** | 33771.52 毫秒 (#29) | 2026年10月23日 |
| DAX 完整构建，2026-08-28 | **39.3 秒（第 1 条，共 19 条）** | 81.2 秒（#13）| 2026-08-28 |
| DAX 完整版本，2026 年 10 月 23 日 | **38.8 秒（第 3 条，共 22 条）** |没有结果 | 2026年10月23日 |

来源：ComputeSDK，通过[我们的基准页面](/docs/benchmarks)。突发启动是指从创建请求到新沙箱内第一个成功命令（少于 100 个并发创建）之间的时间。如果未列出某个运行，则表示该提供程序未出现在该运行中。

> **注意：公平阅读**
>
> 这里的突发数是在重并发下； CodeSandbox 是一个成熟的交互式开发产品，但本基准测试并不能完全代表这一点。

## 并排的功能

每个 CodeSandbox 答案都来自 CodeSandbox 自己的文档，链接在最后一列中。 codesandbox.io 阻止自动获取，因此这些答案是从 2026 年 4 月 21 日的存档副本中读取的，可能已过时。
每个 MIOSA 答案都链接了记录它的页面。
“未记录”意味着供应商的文档没有说明，而不是缺少该功能。

|能力|MIOSA |代码沙盒 |来源 |
|---|---|---|---|
|隔离| [默认情况下 Firecracker microVM](/docs/develop/sandboxes) |爆竹 microVM | [CodeSandbox 文档](https://codesandbox.io/docs/sdk/core-concepts) |
|沙箱寿命 | 【空闲超时，默认1小时，最长24小时，可延长；计算机持续存在](/docs/develop/sandboxes/timeouts) | 1 分钟至 24 小时不活动后进入休眠状态 | [CodeSandbox 文档](https://codesandbox.io/docs/sdk/hibernate) |
|暂停、快照、分叉 | [是：暂停和恢复、快照、恢复和分叉](/docs/develop/snapshots) |是：使用内存休眠和恢复，分叉 | [CodeSandbox 文档](https://codesandbox.io/docs/sdk/hibernate) |
|完整的Linux桌面和计算机使用| [是的，计算机](/docs/computers/overview) |没有记录| [CodeSandbox 文档](https://codesandbox.io/docs/sdk) |
|端口的公共预览 URL | [是](/docs/develop/previews) |是的 | [CodeSandbox 文档](https://codesandbox.io/docs/sdk/hosts) |
|部署到生产 URL | [是的，服务和 Docker 主机，具有发布和回滚](/docs/services/overview) |没有记录| [CodeSandbox 文档](https://codesandbox.io/docs/sdk) |
|托管数据库| [Postgres、MySQL、Redis、MongoDB、Qdrant](/docs/data/overview) |没有记录| [CodeSandbox 文档](https://codesandbox.io/docs/sdk) |
| SDK | [TypeScript、Python、Go、Elixir、Java、Rust](/docs/sdks) | JavaScript/TypeScript | [CodeSandbox 文档](https://codesandbox.io/docs/sdk) |
|命令行 | [是](/docs/cli) |是的 | [CodeSandbox 文档](https://codesandbox.io/docs/sdk/cli) |
| MCP服务器| [是，托管](/docs/mcp) |没有记录| [CodeSandbox 文档](https://codesandbox.io/docs/sdk) |
|自带云 | [是的，AWS 和 Google Cloud](/docs/infrastructure/byoc/overview) |没有自托管 | [CodeSandbox 文档](https://codesandbox.io/docs/sdk/faq) |
|计费| [运行时每秒，每种尺寸一个价格](/docs/pricing) |每分钟，以 VM 积分为单位 | [CodeSandbox 文档](https://codesandbox.io/docs/sdk/pricing) |
|地区 | [美国东部、美国西部、美国米亚](/docs/regions) |没有记录| [CodeSandbox 文档](https://codesandbox.io/docs/sdk) |

## MIOSA 的不同之处

本网站上记录了 MIOSA 的这些功能：

- **具有完整 Linux 桌面的持久计算机**，而不仅仅是无头沙箱。请参阅[计算机](/docs/computers/overview)。
- **部署和 Docker 主机**，因此机器成为版本化 URL 或容器化应用程序。请参阅[部署](/docs/docker-host/overview)。
- **白标平台**，让您的客户看到您的品牌和域名。请参阅[白标签设置](/docs/platform/white-label)。
- **适合所有人的一个界面**：MCP、CLI 和六个 SDK。请参阅 [MCP](/docs/mcp)、[CLI](/docs/cli)、[SDK](/docs/sdks)。
- **具有连接注入的托管数据**：Postgres、MySQL、Redis、MongoDB、Qdrant。请参阅[数据](/docs/data/overview)。
- **区域和 OpenComputers** 适用于您拥有的机器。请参阅[区域](/docs/regions)和[OpenComputers](/docs/opencomputers)。

## 何时选择每个- **如果您想要为人员和智能体提供即时、交互式云开发环境，请选择 CodeSandbox**。
- **如果您想要为智能体和应用程序构建运行时，请选择 MIOSA**：沙箱、计算机、智能体运行、部署和托管数据，以及上面测量的启动数据。

## 来源

- CodeSandbox 公共文档：https://codesandbox.io（截至 2026 年 10 月）。
- ComputeSDK 基准测试结果，来自[我们的基准测试页面](/docs/benchmarks)（2026 年 10 月 23 日）。
- MIOSA 功能，如本网站所述。

- [所有比较](/docs/compare)：对每个提供商 MIOSA 进行衡量。
