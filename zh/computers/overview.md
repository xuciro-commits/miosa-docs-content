---
slug: "computers/overview"
title: "电脑（台式机）"
description: "适用于浏览器智能体、人工接管、GUI 自动化和长期运行的计算机使用工作流程的耐用云桌面。"
section: "Computers"
order: 3000
---

**计算机**是耐用的云桌面。
它为每个用户、智能体或工作流程提供了一个真正的 Linux GUI，其中包含浏览器状态、文件、shell 访问、桌面流和编程控制。
当工作依赖于像素、会话、浏览器登录、桌面应用程序、可访问性树或人工接管智能体正在使用的同一台机器时，请使用它。

对于纯代码执行、构建和预览服务器，[沙箱](/docs/develop/sandboxes) 更便宜、更快。
对于 24/7 生产 Web 应用程序，请使用 [Docker Host](/docs/docker-host/docker-deploy)。
还在决定吗？请参阅[计算机与沙箱](/docs/computers/vs-sandbox)。

## 1. 创建一台计算机

**命令行界面**

```bash
miosa computer create --size small
miosa computer list
```

**打字稿**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const computer = await miosa.computers.create({
  name: "agent-1",
  template_type: "miosa-desktop",
  size: "small",
})
await computer.start()
```

**Python**

```python
import miosa, os

client = miosa.Miosa(api_key=os.environ["MIOSA_API_KEY"])

computer = client.computers.create(
    name="agent-1",
    template_type="miosa-desktop",
    size="small",
)
computer.start()
```

传递 `external_workspace_id` 将计算机归属于您的租户之一。
有关状态、大小和工作区范围，请参阅[生命周期和大小调整](/docs/computers/lifecycle)。

## 2. 驱动桌面

**命令行界面**

```bash
miosa api GET /computers/cmp_123/desktop/screenshot > screen.png
```

**打字稿**

```typescript
const png = await computer.screenshot()
await computer.leftClick(640, 480)
await computer.type("hello world")
await computer.key("Return")
await computer.bash("firefox &")
```

**Python**

```python
png_bytes = computer.screenshot()
computer.left_click(640, 480)
computer.type("hello world")
computer.key("Return")
computer.bash("firefox &")
```

以观察开始每个循环，执行一次，然后再次观察。
完整的操作集位于[计算机使用](/docs/computer-use)中。

## 3.观看直播

计算机也是实时像素流。
从 MIOSA 控制台打开它，或者使用后端创建的短期 URL 将相同的流嵌入到您的应用程序中。

```typescript
const urls = await computer.urls()
console.log(urls.desktop_url)
```

有关令牌和 iframe 流程，请参阅[嵌入和流式传输](/docs/computers/embedding)。

## 选择你需要的

|你想要|从这里开始 |
|---|---|
|创建、调整和管理机器 | [生命周期和大小调整](/docs/computers/lifecycle) |
|选择计算机或沙箱 | [计算机与沙箱](/docs/computers/vs-sandbox) |
|看看盒子里装了什么 | [计算机内部](/docs/computers/anatomy) |
|从代码驱动桌面 | [计算机使用](/docs/computer-use) |
|将实时桌面嵌入到您的 UI 中 | [嵌入和流式传输](/docs/computers/embedding) |
|安全运送 | [生产清单](/docs/computers/production-checklist) |
|测量启动 | [快速启动和准备就绪](/docs/computers/fast-start) |

## 下一步

- [计算机与沙箱](/docs/computers/vs-sandbox)：根据经验选择适合工作负载的机器。
- [计算机内部](/docs/computers/anatomy)：包装盒中包含哪些内容以及桌面、磁盘、流和 API 如何组合在一起。
- [生命周期和大小调整](/docs/computers/lifecycle)：快速启动、状态机、资源大小和工作区范围的创建。
- [嵌入和流媒体](/docs/computers/embedding)：访问模型、操作员工作流程以及将实时桌面嵌入到您的 UI 中。
- [生产清单](/docs/computers/production-checklist)：在生产中依赖计算机之前要验证的默认值。
- [计算机使用](/docs/computer-use)：完整操作参考：屏幕截图、单击、键入、按键、滚动、拖动、窗口、辅助功能树。
