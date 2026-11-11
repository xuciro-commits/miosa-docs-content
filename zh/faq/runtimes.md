---
slug: "faq/runtimes"
title: "SOMA 和运行时"
description: "SOMA 是什么、默认运行时、如何选择、访问请求和可用性。"
section: "Help"
order: 16005
---

关于启动沙箱的运行时的简短答案。

## 什么是 SOMA？

SOMA（安全优化机器架构）是 MIOSA 在 Linux KVM 上自己的开源 Rust VMM，为每个沙箱提供一个具有自己内核的硬件隔离 VM。
请参阅[SOMA](/docs/soma)。

## 如何选择运行时？

创建沙箱时，从 REST API、SDK (`runtime`)、CLI (`miosa create --runtime`) 或控制台传递 `runtime_profile: "standard" | "soma" | "auto"`。
无效值返回 `INVALID_SANDBOX_RUNTIME_PROFILE`；没有无声的后备措施。
`GET /api/v1/sandboxes/runtimes` 列出了您的组织可以选择的运行时。
请参阅[选择运行时](/docs/soma/choosing-a-runtime)。

```bash
miosa create --runtime soma --size small
```

## SOMA 适合所有人吗？

是的。 SOMA 可在 `small`、`medium` 和 `large` 上供每位客户使用，并可实时调整 vCPU 和内存大小。
`GET /api/v1/sandboxes/runtimes` 报告当前每个运行时支持的内容。
请参阅[SOMA](/docs/soma)。

## 什么是正式可用的，什么是预览版？

沙盒、预览、发布、数据、白标签、SOMA 和沙盒浏览器通常可用。
计算机、动态部署、Docker Host 和租户部署域处于合作伙伴预览版； GPU 支持的沙箱尚未产品化，仍保留在路线图上。
请参阅[可用性和路线图](/docs/availability)。
