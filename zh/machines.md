---
slug: "machines"
title: "机器能力"
description: "每台机器附带的内容、尺寸、运行时间和预装工具。"
section: "Sandboxes"
order: 2013
---

机器是沙箱或计算机背后的计算：版本化模板、命名大小、运行时引擎以及映像中已安装的工具。
选择最接近您的堆栈的模板，选择大小，然后 MIOSA 启动机器准备运行您的代码。

## 模板

**模板**是已安装特定语言、框架或工具链的预构建沙箱映像。
选择与您的堆栈最接近的匹配项；如果需要，您可以覆盖每个沙箱的安装/启动命令。
有关内置图像和自定义 BuildSpec 参考，请参阅[模板](/docs/develop/templates)。

## 尺寸

每个沙箱都在命名资源合约上运行。
默认值为 `small`：2 个 vCPU、4096 MiB RAM 和 10240 MiB 磁盘。
优先选择命名的 `size` 而不是原始 CPU、内存和磁盘字段。
请参阅[规模和限制](/docs/develop/sandboxes/sizing)了解合同以及每个合同的费用。

## 运行时

沙箱在两个引擎之一上运行：**Firecracker**（MIOSA 的长期默认引擎）或 **SOMA**（MIOSA 自己的开源 VMM）。
您的组织选择它提供的运行时，沙箱记录它创建时使用的运行时。
请参阅[选择运行时](/docs/soma/choosing-a-runtime)列出您可以选择的运行时以及它目前支持的每个报告。

## 预装工具

模板附带的工具链已在映像中，因此在执行第一个命令之前无需安装任何内容。
带有 `docker compose` 和 `docker buildx` 插件的 Docker 在每个沙箱中都可用，包括 SOMA 运行时。

## 下一步

- [模板](/docs/develop/templates)：内置图像以及每个图像的预构建用途，以及自定义构建规格。
- [大小调整和限制](/docs/develop/sandboxes/sizing)：命名资源合同、原始覆盖和账单说明。
- [选择运行时](/docs/soma/choosing-a-runtime)：Firecracker、SOMA 或 auto，以及如何列出您的组织提供的服务。
- [沙箱](/docs/develop/sandboxes)：完整的沙箱产品：执行、文件、预览、快照和生命周期。
