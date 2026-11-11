---
slug: "develop/sandboxes"
title: "沙箱"
description: "用于代码执行、构建、测试和开发服务器的快速、隔离的 Linux VM，具有实时预览 URL 和快照。"
section: "Sandboxes"
order: 2000
---

**沙箱**是 MIOSA 的快速代码执行环境。
您的智能体将文件写入其中，执行命令，将端口公开为实时预览 URL，以及用于分支或回滚的快照状态。
将其用于构建循环、测试、脚本、包安装、代码生成、预览服务器和短期自动化。

默认情况下，沙箱是持久的，这意味着它们可以在停止后继续存在。
当您停止其中一个或其超时到期时，MIOSA 会保留磁盘并将沙箱移至 `stopped`，以便稍后可以恢复。
正在运行的进程无法在停止后继续存在。
持久并不意味着沙箱在失去其主机后仍然存在。请参阅[沙盒生命周期](/docs/develop/sandboxes/lifecycle)。
SOMA 尚不支持持久化。
仅当应删除保存的状态时才销毁。

## 1.创建沙箱

**命令行界面**

```bash
miosa create my-box --wait     # provision a sandbox and wait until it is ready
```

**打字稿**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const sandbox = await miosa.sandboxes.create()
console.log(sandbox.id, sandbox.state)
// sbx_a1b2c3d4e5  running
```

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sandbox = miosa.sandboxes.create()
print(sandbox.id, sandbox.state)
# sbx_a1b2c3d4e5  running
```

## 2. 运行命令

**命令行界面**

```bash
miosa exec my-box -- echo "hello from MIOSA"
```

**打字稿**

```typescript
const result = await sandbox.exec("echo hello from MIOSA")
console.log(result.stdout)
// hello from MIOSA
```

**Python**

```python
result = sandbox.exec("echo hello from MIOSA")
print(result.stdout)
# hello from MIOSA
```

## 3.写入文件并打开预览

**命令行界面**

```bash
echo "<h1>Hello</h1>" | miosa files write my-box:/workspace/index.html
miosa preview create my-box 3000 --visibility public
```

**打字稿**

```typescript
await sandbox.files.write("/workspace/index.html", "<h1>Hello</h1>")
const preview = await sandbox.previews.create(3000)
console.log(preview.url)
// https://3000-abc12345.sandbox.miosa.ai
```

**Python**

```python
sandbox.files.write("/workspace/index.html", "<h1>Hello</h1>")
preview = sandbox.previews.create(port=3000)
print(preview.url)
# https://5173-abc12345.sandbox.miosa.ai
```

打开 MIOSA 返回的预览 URL。只有您公开的端口才能访问，并且只能通过 MIOSA 智能体访问。

> **注意：规则**
>
> 对代码和流程使用沙箱。
> 使用 [计算机](/docs/computers/overview) 进行可见 GUI、浏览器登录或人工接管。
> 当结果应该是具有自定义域的持久 24/7 部署 URL 时，请使用 [Docker Host](/docs/docker-host/docker-deploy)。
> 每个沙箱都启动相同的主映像，该映像附带了 Docker 和 `docker compose` 和 `docker buildx` 插件，以及虚拟显示器上带有 CDP 的实时浏览器 (`Xvfb :99`)，因此您可以在开发时构建和运行容器，并驱动浏览器。
> SOMA 沙箱还没有该显示，因此实时查看器在那里不可用。

## 图像、模板和运行时

三个不同的因素决定了沙盒靴子的内容，并且它们不是相同的选择。

| |它是什么 |在哪里设置 |
|---|---|---|
| **图片** |一个主映像，包含 Node、Python、带有 Compose 和 Buildx 的 Docker Engine 29.9、带有 CDP 的实时浏览器以及内置的代码解释器。 |没有什么可以选择的：每个沙箱都会启动它。 |
| **模板** |创建时在该映像内暂存的可选内容：文件、脚手架、包和启动命令。 | `template_id`，例如 `nextjs`、`static-html` 或自定义模板。 |
| **运行时** |哪个虚拟机监视器运行沙箱。 | `runtime`：`firecracker`（默认）、`soma` 或 `auto`。 |

模板从来都不是一个单独的图像。
作为 `template_id` 传递的旧映像 ID 仍然有效并启动主映像；请参阅[沙盒模板 API](/docs/api-reference/sandbox-templates)。
请参阅[模板](/docs/develop/templates)了解应用程序启动器，并[选择运行时](/docs/soma/choosing-a-runtime)了解运行时选项。

## 选择你需要的

|需要|产品 |结果 |
|---|---|---|
|运行命令、编辑文件、安装包、测试代码 | **沙盒** |具有执行、文件、快照和预览 URL 的隔离运行时 |
|使用浏览器登录、桌面应用程序、屏幕截图、鼠标、键盘 | **计算机** |具有流媒体和桌面控制功能的耐用 GUI 机器 |
|为用户发布 24/7 应用程序 | **Docker 主机** |具有自定义域支持的始终在线部署 URL |

沙箱非常适合构建和预览。
它们不是最终的生产托管层。
当预览版成为面向用户的应用程序时，通过 Docker Host 发布它，以便它获得正确的路由、域、生命周期和计费行为。

## 下一个

- [创建和连接](/docs/develop/sandboxes/create)：融合运行、完整创建、上下文管理器以及连接到现有沙箱。
- [生命周期](/docs/develop/sandboxes/lifecycle)：状态机，每个状态的含义以及如何暂停和恢复。
- [大小调整和限制](/docs/develop/sandboxes/sizing)：指定资源合同、账单注释和选择加入的运行程序快速路径。
- [超时和自动停止](/docs/develop/sandboxes/timeouts)：默认 TTL、扩展它以及如何测量使用情况。
- [沙盒事件](/docs/develop/sandboxes/events)：通过 SSE 订阅生命周期和活动事件。
- [生产清单](/docs/develop/sandboxes/production-checklist)：在生产中依赖沙箱之前设置的默认值。
- [环境变量](/docs/develop/sandboxes/environments)：在创建时设置和读取环境变量，以及秘密应该存在的位置。
- [功能](/docs/develop/sandboxes/capabilities)：沙箱的用途、沙箱的用途以及核心功能。

- [沙箱中的 Docker](/docs/develop/sandboxes/docker)：沙箱内的 Docker Engine、Compose 和 Buildx。
- [连接到正在运行的沙箱](/docs/develop/sandboxes/connect)：SDK、CLI、通过隧道的 SSH 和预览 URL。
- [互联网访问和出口](/docs/develop/sandboxes/internet-access)：默认网络行为和白名单。
- [长时间运行的任务](/docs/develop/sandboxes/long-running-tasks)：后台进程和永远在线的沙箱。
- [元数据和标签](/docs/develop/sandboxes/metadata)：附加和读取任意元数据。

## 相关

- [Files & Exec](/docs/develop/files-and-exec)：完整的文件系统和进程 API：递归操作、监视、分离作业。
- [预览](/docs/develop/previews)：通过共享令牌和可见性控制智能体到沙箱端口的实时 HTTPS URL。
- [模板](/docs/develop/templates)：内置图像和自定义 BuildSpec 参考。
- [快照和持久性](/docs/develop/snapshots)：暂停、恢复、分叉和当前公共快照边界。
