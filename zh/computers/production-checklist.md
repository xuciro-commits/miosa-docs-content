---
slug: "computers/production-checklist"
title: "生产清单"
description: "在生产中依赖计算机之前要验证的默认设置：工作区归属、访问、机密、状态、可观察性和恢复。"
section: "Computers"
order: 3006
---

在生产中依赖计算机之前，请验证这些要点。

|面积 |检查 |
|---|---|
|工作空间归属|构建白标签应用程序时设置 `external_workspace_id`、`external_user_id` 或工作区/项目字段 |
|访问 |为普通用户使用平台身份验证或流门票 |
|秘密|将 API 密钥放在后端代码或加密环境中，而不是放在前端包中 |
|状态 |对持久会话使用停止/启动，并仅在应删除状态时进行删除 |
|可观察性|一起记录计算机 ID、外部 ID 和任务/运行 ID |
|恢复|设计应用程序，以便用户可以在刷新浏览器后重新打开同一台正在运行的计算机 |

> **注意：相关**
>
> [生命周期和大小调整](/docs/computers/lifecycle)涵盖这些检查假设的状态和大小，[嵌入和流媒体](/docs/computers/embedding)涵盖访问模型和流票证。
