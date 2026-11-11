---
slug: "faq/sandboxes-and-computers"
title: "沙箱和计算机"
description: "沙箱与计算机、默认沙箱、运行长度、状态、暂停和快照。"
section: "Help"
order: 16002
---

## 沙箱还是计算机？

对代码、文件和进程使用 [沙箱](/docs/develop/sandboxes)。
当您需要可见桌面、浏览器登录或人工接管时，请使用 [计算机](/docs/computers/overview)。
当您需要持久、始终在线的 URL 时，请使用 **Docker Host**。

```bash
miosa create my-box --wait     # provision a sandbox and wait until it is ready
miosa list                     # see all your sandboxes
```

## 我在默认沙箱中能得到什么？

默认情况下持久的无头 microVM，以 `/workspace` 作为可编辑根，从模板（通常 `miosa-sandbox` 启动）。
默认大小 `small` 为 2 个 vCPU、4096 MiB RAM 和 10240 MiB 磁盘，并且响应携带版本化的 `resource_contract`。
请参阅[沙箱](/docs/develop/sandboxes)和[大小调整和限制](/docs/develop/sandboxes/sizing)。

## 计算机有什么不同？

计算机是一个耐用的云 Linux 桌面（Xfce 和 Firefox），具有屏幕截图、点击、输入和流式传输以及人工接管功能。
您可以通过`computer.urls()["desktop_url"]`嵌入或共享它。
请参阅[计算机](/docs/computers/overview)和[嵌入实时桌面](/docs/computers/embedding)。

## 沙箱可以运行多长时间？

`timeout_sec` 为 1 到 86400 秒，默认为 3600。
`always_on` 根据租户策略禁用超时强制执行。
超时从最后一个活动开始计算，过期会停止持久性沙箱并销毁非持久性沙箱。
默认情况下，对于 2026 年 10 月 11 日或之后创建的帐户，沙箱也会在闲置 30 分钟后停止（将 `idle_timeout_sec` 设置为 `0` 以禁用），并且停止的沙箱会在下一个请求时唤醒。
请参阅[超时和自动停止](/docs/develop/sandboxes/timeouts)。

## 沙箱状态是什么？

`creating`、`running`、`stopped`、`paused`、`error` 和 `destroyed`。
`paused` 仅存在于通告暂停的运行时，即今天的 Firecracker；索玛没有停顿。
请参阅[沙盒生命周期](/docs/develop/sandboxes/lifecycle)。

## 停止沙箱会丢失我的文件吗？

不会。停止会保留沙箱 ID 和磁盘，包括文件、已安装的软件包和启用的服务；简历将其带回来。
停止永远不会保持正在运行的进程。
暂停（仅限 Firecracker）使进程最多保留 30 分钟，然后就像停止一样。
持久意味着沙箱在停止后仍然存在，而不是主机丢失； `durability` 字段表示有多少主机保持已保存的状态。
SOMA 尚不支持持久性、暂停或快照。
销毁是删除保存的状态。
请参阅[快照和持久性](/docs/develop/snapshots)。

## 快照和分叉有什么区别？

Restore从检查点构建一个新的沙箱； fork 将一个正在运行的沙箱分支为一个独立的沙箱。
两者都保持源不变。
请参阅[快照和持久性](/docs/develop/snapshots)。

```bash
miosa snapshot save my-box before-upgrade   # pin a named restore point
miosa snapshot fork before-upgrade          # branch it into an independent sandbox
```

## 快照会保存我的数据库吗？

不会。快照捕获沙箱磁盘，并且仅捕获内存（在运行时公布的情况下）。
外部数据库、对象存储、连接器和第三方服务不会随之回滚。
在数据服务中保持持久状态。
请参阅[数据概述](/docs/data/overview)。
