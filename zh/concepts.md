---
slug: "concepts"
title: "核心概念"
description: "MIOSA 的标准产品模型：从可变的沙箱到不可变的发布、稳定的部署、持久数据，以及客户自有的基础设施。"
section: "Optional"
order: 17005
---

MIOSA 为开发人员和智能体商提供了安全的开发循环，而不会使生产发生变化。
您在沙箱中工作，通过预览检查结果，并发布冻结的候选版本。
MIOSA 验证该候选版本后，升级会将部署的稳定 URL 移至新版本。

如果您只阅读一行，请记住这一点：

**沙箱（可变）→预览→发布→发布（不可变）→验证→升级→部署（稳定URL）。**

旧版本仍然可以回滚。
数据库、存储、机密和域仍然附加到部署，而不是复制到每个版本中。

> **注：中央边界**
>
> 更改沙箱会改变开发状态。
> 它不会改变生产。
> 仅当提升确切的不可变候选者时，生产才会发生变化。

此页面是**资源模型**：MIOSA 资源是什么以及它们如何关联。
对于 **API 协议** - 身份验证、令牌类型、幂等性、速率限制、错误和事件 - 请参阅 [API 基础知识 ](/docs/fundamentals)。

## 选择你需要的

|你想要|阅读 |
|---|---|
|所有权链以及开发和生产资源| [资源模型](/docs/concepts/resource-model) |
|发布如何成为实时发布 | [发布管道](/docs/concepts/publish-pipeline) |
|数据库、存储和绑定 | [数据和绑定](/docs/concepts/data-and-bindings) |
|桌面产品| [计算机](/docs/concepts/computers-desktop) |
|白标租赁和归属| [租赁](/docs/concepts/tenancy) |
|用于预览和共享的浏览器令牌 | [预览并分享代币](/docs/concepts/tokens) |
|租户预览域名和品牌 | [租户配置](/docs/concepts/tenant-config) |

对平台词汇不熟悉？
当您遵循下面的资源和发布流程时，请保持 [MIOSA 术语表](/docs/glossary) 处于打开状态。

## 下一步

- [资源模型](/docs/concepts/resource-model)：所有权链、沙箱和预览 - 平台的可变方面。
- [发布管道](/docs/concepts/publish-pipeline)：构建器、发布、运行时实例、域、部署和版本：源如何成为实时发布。
- [数据和绑定](/docs/concepts/data-and-bindings)：Postgres、Redis、对象存储、卷、身份验证以及它们如何附加到部署。
- [计算机](/docs/concepts/computers-desktop)：云桌面产品，以及计算机和沙箱之间的区别。
- [租赁](/docs/concepts/tenancy)：基于 MIOSA 构建的平台的白标租赁和外部归因。
- [令牌](/docs/concepts/tokens)：预览令牌并共享浏览器访问沙盒预览的令牌。
- [租户配置](/docs/concepts/tenant-config)：租户预览域和租户品牌。

## 深入研究

**计算机（桌面产品）** 现在位于 [Computers](/docs/concepts/computers-desktop)。

**租户预览域**现在位于[租户配置](/docs/concepts/tenant-config)中。
