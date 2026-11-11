---
slug: "concepts/resource-model"
title: "资源模型"
description: "MIOSA 所有权链和可变的开发资源 - 组织、工作空间、项目、沙箱和预览。"
section: "Optional"
order: 17006
---

此页面是 MIOSA 可变方面的**资源模型**：所有权链和开发资源。
有关生产资源和发布管道，请参阅[发布管道](/docs/concepts/publish-pipeline)。

## 资源层次结构

MIOSA 通过清晰的所有权链解决每项操作：

```text
Organization → Workspace → Project → Sandbox or Deployment
```

- **组织**是顶级客户或平台帐户。
- **工作空间**隔离团队、客户或业务边界。
- **项目**将一种产品的开发和生产资源分组。
- **沙盒**是工作发生的可变场所。
- **部署**是拥有版本、路由和绑定的持久生产身份。

每个突变都必须解决一个明确的组织、工作空间、项目和目标资源。
这可以防止智能体或 CLI 命令发布到错误的客户或环境。

## 开发资源

### 沙盒

**沙盒**是一个可变的、隔离的环境，智能体或开发人员可以在其中构建应用程序。
它包括一个完整的开发环境，包括 Node、Python、git、包管理器、文件 API 和终端。

主要特性：

- **可变且有状态。** `/workspace` 中的文件在命令中持续存在，直到沙箱被破坏或暂停。
- **可执行。** 运行 `npm install`、`python app.py` 或 `git clone` 等命令，并具有结构化的 stdout、stderr 和退出状态。
- **智能体友好的文件访问。** 读取、写入和同步源、配置和资产。
- **可预览。** 通过临时预览 URL 公开正在运行的端口。
- **快照感知。** 保存、分叉和恢复开发状态，而不将该快照视为生产版本。
- **可挂起。** 空闲计算可以暂停，稍后从持久状态恢复。

沙盒不是生产。
它的文件可以随时更改，它的进程可以停止，并且它的预览可以在暂停时消失。
使用它来创建和测试将成为生产候选的源。

```bash
miosa create my-box --wait     # provision a sandbox and wait until it is ready
miosa exec my-box -- echo "hello from MIOSA"
```

深入研究沙箱、文件 I/O、执行、预览和开发循环。
[沙箱参考 →](/docs/develop/sandboxes)

### 预览

**预览版**是一种临时公共 URL，可通过隧道连接到正在运行的沙箱内的一个端口。

```text
https://5173-abc12345.sandbox.miosa.ai
```

创建预览时，指定端口，通常为 `3000` 或 `5173`。
MIOSA 将该主机名路由到沙箱进程。
随着文件的更改和开发服务器的重新加载，预览也会发生变化。

主要特性：

- **可变。** 每个请求都反映当前的沙盒进程和文件。
- **短暂的。** URL 取决于正在运行的沙箱，并且不是持久的客户 URL。
- **可共享。** 短期令牌允许浏览器访问，而无需暴露工作区 API 密钥。

```bash
miosa preview create my-box 3000 --visibility public
```

[预览参考→](/docs/develop/previews)

## 下一步

- [发布管道](/docs/concepts/publish-pipeline)：构建器、发布、运行时实例、域、部署和版本。
- [数据和绑定](/docs/concepts/data-and-bindings)：比沙箱寿命更长的持久资源。
- [核心概念](/docs/concepts)：一屏概述。
