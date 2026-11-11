---
slug: "compare/daytona"
title: "MIOSA vs 代托纳"
description: "使用截至 2026 年 10 月的独立 ComputeSDK 基准测试和 Daytona 的公共文档对 MIOSA 和 Daytona 进行事实比较。"
section: "Optional"
order: 17018
---

比较

Daytona 和 MIOSA 都在隔离的沙箱中运行人工智能生成的代码。本页在独立的 ComputeSDK 基准测试和 MIOSA 功能文档上对它们进行了比较。

## 代托纳是什么

Daytona 提供了用于运行 AI 生成的代码的沙箱和基础设施，其公共文档（[daytona.io/docs](https://www.daytona.io/docs)，截至 2026 年 10 月）对此进行了描述。

## 测量性能

独立的 ComputeSDK 基准测试针对每个提供商运行相同的工作负载。这是这里唯一可以比较的数字。

|公制|MIOSA |代托纳 |截至 |
|---|---|---|---|
|突发启动、中位交互时间 | **57.09 毫秒（第 1 条，共 32 条）** | 1094.64 毫秒 (#19) | 2026年10月23日 |
| DAX 完整构建，2026-08-28 | **39.3 秒（第 1 条，共 19 条）** | 141.3 秒 | 2026-08-28 |
| DAX 完整版本，2026 年 10 月 23 日 | **38.8 秒（第 3 条，共 22 条）** | 79.5 秒 (#15) | 2026年10月23日 |

来源：ComputeSDK，通过[我们的基准页面](/docs/benchmarks)。突发启动是指从创建请求到新沙箱内第一个成功命令（少于 100 个并发创建）之间的时间。如果未列出某个运行，则表示该提供程序未出现在该运行中。

> **注意：公平阅读**
>
> 启动和构建时间是负载下的两个指标。 Daytona 的构建时间在最新运行中缩短至 70.0 秒，这些数字并未反映 API 设计、定价或适配情况。

## 并排的功能

每个 Daytona 答案均来自 Daytona 自己的文档或定价页面，链接在最后一列中，获取日期为 2026 年 10 月 23 日。
每个 MIOSA 答案都链接了记录它的页面。
“未记录”意味着供应商的文档没有说明，而不是缺少该功能。

|能力|MIOSA |代托纳 |来源 |
|---|---|---|---|
|隔离| [默认情况下 Firecracker microVM](/docs/develop/sandboxes) |容器或完整虚拟机，按沙箱类别 | [代托纳文档](https://www.daytona.io/docs/en/isolation.md) |
|沙箱寿命 | 【空闲超时，默认1小时，最长24小时，可延长；计算机持续存在](/docs/develop/sandboxes/timeouts) |默认15分钟后自动停止，可禁用 | [代托纳文档](https://www.daytona.io/docs/en/sandboxes.md) |
|暂停、快照、分叉 | [是：暂停和恢复、快照、恢复和分叉](/docs/develop/snapshots) |是：虚拟机上的快照、暂停、恢复和分叉 | [代托纳文档](https://www.daytona.io/docs/en/persistence.md) |
|完整的Linux桌面和计算机使用| [是的，计算机](/docs/computers/overview) |是：Linux、Windows 和 macOS 计算机使用 | [代托纳文档](https://www.daytona.io/docs/en/computer-use.md) |
|端口的公共预览 URL | [是](/docs/develop/previews) |是的 | [代托纳文档](https://www.daytona.io/docs/en/preview.md) |
|部署到生产 URL | [是的，服务和 Docker 主机，具有发布和回滚](/docs/services/overview) |没有记录| [代托纳文档](https://www.daytona.io/docs/en/) |
|托管数据库| [Postgres、MySQL、Redis、MongoDB、Qdrant](/docs/data/overview) |没有记录| [代托纳文档](https://www.daytona.io/docs/en/) |
| SDK | [TypeScript、Python、Go、Elixir、Java、Rust](/docs/sdks) | Python、TypeScript、Ruby、Go、Java | [代托纳文档](https://www.daytona.io/docs/en/) |
|命令行 | [是](/docs/cli) |是的 | [代托纳文档](https://www.daytona.io/docs/en/) |
| MCP服务器| [是，托管](/docs/mcp) |是的 | [代托纳文档](https://www.daytona.io/docs/en/mcp.md) |
|自带云 | [是的，AWS 和 Google Cloud](/docs/infrastructure/byoc/overview) |是的，您自己的跑步者的自定义区域 | [代托纳文档](https://www.daytona.io/docs/en/bring-your-own-compute.md) |
|计费| [运行时每秒，每种尺寸一个价格](/docs/pricing) |每秒、每个 vCPU 和 GiB | [代托纳文档](https://daytona.io/pricing) |
|地区 | [美国东部、美国西部、美国米亚](/docs/regions) |美国和欧盟，以及专用和定制区域 | [代托纳文档](https://www.daytona.io/docs/en/regions.md) |

## MIOSA 的不同之处

本网站上记录了 MIOSA 的这些功能：

- **具有完整 Linux 桌面的持久计算机**，而不仅仅是无头沙箱。请参阅[计算机](/docs/computers/overview)。
- **部署和 Docker 主机**，因此机器成为版本化 URL 或容器化应用程序。请参阅[部署](/docs/docker-host/overview)。
- **白标平台**，让您的客户看到您的品牌和域名。请参阅[白标签设置](/docs/platform/white-label)。
- **适合所有人的一个界面**：MCP、CLI 和六个 SDK。请参阅 [MCP](/docs/mcp)、[CLI](/docs/cli)、[SDK](/docs/sdks)。
- **具有连接注入的托管数据**：Postgres、MySQL、Redis、MongoDB、Qdrant。请参阅[数据](/docs/data/overview)。
- **区域和 OpenComputers** 适用于您拥有的机器。请参阅[区域](/docs/regions)和[OpenComputers](/docs/opencomputers)。

## 何时选择每个- **如果您想要一个专注的沙箱和开发人员环境平台，并且您的产品不需要桌面、部署或白标租户界面，请选择 Daytona**。
- **如果您想要将代码沙箱、完整的计算机、智能体运行、部署、托管数据和品牌平台集中在一处，请选择 MIOSA**，并附上上面测量的启动数据。

## 来源

- Daytona 公共文档：https://www.daytona.io/docs（截至 2026 年 10 月）。
- ComputeSDK 基准测试结果，来自[我们的基准测试页面](/docs/benchmarks)（2026 年 10 月 23 日）。
- MIOSA 功能，如本网站所述。

- [MIOSA vs Modal](/docs/compare/modal)：下一个比较。
- [所有比较](/docs/compare)：对每个提供商 MIOSA 进行衡量。
