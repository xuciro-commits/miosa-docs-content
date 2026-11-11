---
slug: "sandboxes-or-computers"
title: "沙箱还是计算机？"
description: "选择应用或智能体在 MIOSA 上的运行位置：需要代码、预览和内置浏览器时用沙箱，需要完整桌面时用计算机，需要发布上线时用 Docker Host。"
section: "Get started"
order: 6
---

大多数工作从**沙箱**开始。
当有人需要看到并使用真实屏幕时，选择**计算机**。
当结果需要持续在线供用户访问时，用 **Docker Host** 发布。

- [沙箱](/docs/develop/sandboxes)：一台快速、隔离的代码机器。可以运行命令、编辑文件、安装软件包、打开实时预览 URL，并驱动内置浏览器。空闲时暂停，下一次请求时唤醒。
- [计算机](/docs/computers/overview)：耐用的云桌面。真实的 Linux 屏幕，带浏览器、桌面应用、鼠标和键盘控制，可供人工接管的实时视图，以及跨会话保留的状态。

**准备上线？** [Docker Host](/docs/docker-host/docker-deploy) 让您构建的内容以稳定 URL 在线运行，并支持自有域名、版本管理和回滚。

## 按需求选择

| 您想要 | 使用 |
|---|---|
| 运行代码、构建、测试或安装软件包 | 沙箱 |
| 在构建过程中展示应用的实时预览 | 沙箱 |
| 让智能体在浏览器中浏览并填写表单 | 沙箱，配合其[内置浏览器](/docs/browser-use) |
| 使用桌面应用，或任何需要完整屏幕的任务 | 计算机 |
| 将登录状态、文件和桌面保留数天 | 计算机 |
| 让人观看并接管智能体的屏幕 | 两者皆可：沙箱浏览器和计算机桌面都提供实时视图 |
| 让应用持续在线供用户使用 | Docker Host |

> **提示：不确定？先从沙箱开始**
>
> 沙箱启动最快、成本最低。
> 只有当工作确实需要桌面时，再改用计算机。

## 创建实例

**CLI**

```bash
# A sandbox for code and previews
miosa create

# A computer with a full desktop
miosa computer create --size small
```

**TypeScript**

```ts
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const sandbox = await miosa.sandboxes.create({ templateId: "miosa-sandbox" });
const computer = await miosa.computers.create({ name: "research-desk", size: "small" });
```

## 通常两者都会用到

典型的智能体产品会在沙箱中编写并运行代码，在任务需要真实桌面时使用计算机，并通过 Docker Host 发布完成的应用。
完整的决策指南参见[智能体设备选择](/docs/guides/agent-device-selection)，其中包括智能体完全不需要机器的情况。

## 下一步

- [快速开始](/docs/quickstart)：几分钟内创建第一个沙箱。
- [计算机](/docs/computers/overview)：生命周期、流式传输和嵌入。
- [构建一个类似……的应用](/docs/build)：选择一款产品，获取分步指南。
