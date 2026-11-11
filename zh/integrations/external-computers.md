---
slug: "integrations/external-computers"
title: "外部计算机提供商"
description: "目前 MIOSA 可以在哪些类型的计算上运行，以及为什么不提供连接第三方沙箱或桌面提供商。"
section: "Self-hosting"
order: 15004
---

MIOSA 默认在自己的云上运行您的工作负载，也可以在您拥有或自行运营的硬件上运行它们。
今天支持两种自带路径。
目前**不**支持连接第三方沙箱或桌面提供商（例如 Orgo、E2B 或 Daytona）：这些适配器不是产品的一部分。

## 计算可以运行的地方

| 来源 | 状态 | 在哪里运行 |
|---|---|---|
| MIOSA 云 | 可用 | MIOSA 管理的沙箱和计算机 |
| OpenComputers | 可用 | 你自己拥有或租用的硬件，通过出站 WebSocket 连接 |
| 自带云 | 可用，需完成接入流程 | 你自己 AWS 或 Google Cloud 帐户内的 Worker 容量 |
| 第三方提供商适配器 | 不提供 | - |

沙箱默认在 Firecracker 上运行。
SOMA 可供每个客户使用，`auto` 让平台选择。
请参阅[选择运行时](/docs/soma/choosing-a-runtime)。

## 在您拥有的硬件上运行

[OpenComputers](/docs/opencomputers) 是 MIOSA 的自带硬件路径。
您在您控制的计算机上运行一个小型主机智能体，它通过出站 WebSocket 向 MIOSA 注册。
然后，该机器向您的应用程序显示与 MIOSA 计算机具有相同的接口，因此您的产品可以驱动两者，而无需在主机上分支。
从[连接主机](/docs/opencomputers/connect)开始。

## 在您自己的云帐户中运行

如果您需要在自己 AWS 或 Google Cloud 帐户内运行 Worker 容量，MIOSA 提供了单独的入职流程。
请参阅[自带云](/docs/infrastructure/byoc/overview)。

## 不提供什么

MIOSA 目前不提供连接第三方计算提供商的适配器。
目前，不支持通过 MIOSA API 驱动 Orgo、E2B、Daytona 或类似机器，并且您无法连接第三方 API 密钥来运行 MIOSA 内的机器。

> **注意：请改用支持的路径**
>
> 如果您需要 MIOSA 云之外的容量，请在您控制的硬件上使用 [OpenComputers](/docs/opencomputers)，或在您自己的云帐户中使用[自带云](/docs/infrastructure/byoc/overview)。

## 另请参阅

- [计算机概述](/docs/computers/overview) - 统一的计算机表面
- [沙箱](/docs/develop/sandboxes) - 隔离的、一次性的代码执行
- [OpenComputers](/docs/opencomputers) - 带上你自己的硬件
- [自带云](/docs/infrastructure/byoc/overview) - 您自己的云帐户中的工作人员容量
- [计算提供商比较](/docs/integrations/comparison) - MIOSA 的费率比较
