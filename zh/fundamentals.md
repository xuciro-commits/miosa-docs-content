---
slug: "fundamentals"
title: "基本概念"
description: "每个 MIOSA 集成商都需要的心理模型 - 工作空间、身份验证、资源、归因、幂等性、速率限制、错误和事件。"
section: "API reference"
order: 12001
---

MIOSA 是一个用于构建和部署应用程序的云平台。在编写第一个 API 调用之前，请阅读此页。它涵盖了支撑每个 MIOSA 集成的概念。

此页面是 **API 协议** 参考：身份验证、令牌、幂等性、速率限制、错误、事件和租户级配置。
对于**资源模型** - 资源是什么以及它们如何关联 - 请参阅[核心概念](/docs/concepts)。

## 每个集成都需要什么

|主题 |阅读 |
|---|---|
| API 密钥、浏览器令牌和五种令牌类型 | [身份验证](/docs/fundamentals/authentication) |
|将资源映射到您自己的客户| [所有权和外部归属](/docs/fundamentals/attribution) |
|安全重试、配额和错误处理 | [请求、限制和错误](/docs/fundamentals/requests) |
|SSE的流媒体进展 | [活动（SSE）](/docs/fundamentals/events) |
| Webhooks 和每最终用户配额 | [租户配置](/docs/fundamentals/tenant-config) |

## 资源所在的地方

本页涵盖 API 协议。完整的**资源模型** - 所有权链、开发和生产资源以及它们之间的关系 - 存在于[核心概念](/docs/concepts)中。

简而言之：您的帐户是一个**组织**（一个永久的 slug、一个计费帐户和 API 密钥）。在其中，**工作空间**分隔客户、客户、团队或环境，而**项目**将一种产品的资源分组。沙箱、计算机、部署、数据库、域、存储、卷、功能和作业都属于一个项目。每个请求都会解决一个明确的组织、工作空间和项目。

## 沙箱与计算机

沙箱和计算机是两种不同的产品，具有不同的生命周期和不同的用例。比较表和资源模型上下文位于 [Core Concepts](/docs/concepts#computers-desktop-product) 中。

- 当智能体需要编写代码、运行 shell 命令、管理文件或为开发服务器提供服务时，使用 **沙箱**。
- 当智能体需要操作图形界面时使用**计算机** - 浏览器、桌面应用程序或任何具有可见屏幕的东西。

这两种产品位于同一组织中，可以按工作区/项目分开，并显示在同一计费帐户中。

**所有权和外部归属**现在位于[所有权和外部归属](/docs/fundamentals/attribution)中。

## 下一步

- [身份验证](/docs/fundamentals/authentication)：API 密钥格式、浏览器令牌以及五种令牌类型及其信任级别。
- [所有权和外部归属](/docs/fundamentals/attribution)：规范所有权字段、外部归属和平台使用情况汇总。
- [请求、限制和错误](/docs/fundamentals/requests)：幂等密钥、速率限制和带有 HTTP 状态代码的错误信封。
- [事件 (SSE)](/docs/fundamentals/events)：通过一次性票证、资源流和租户范围的流来流式传输进度。
- [租户配置](/docs/fundamentals/tenant-config)：白标签集成商的 Webhook 订阅和每最终用户配额。

- [快速入门](/docs/quickstart)：五分钟实践之旅 - 创建沙箱、编写代码、发布到实时 URL。
- [沙箱](/docs/develop/sandboxes)：完整的沙箱生命周期 - 创建、执行、文件、快照、自动挂起。
- [模板](/docs/develop/templates)：为 Node、Next.js、Python、Go、Rust 等预先构建的图像。
- [部署概述](/docs/docker-host/overview)：构建器、发布、版本、运行时实例 - 完整的发布管道。
