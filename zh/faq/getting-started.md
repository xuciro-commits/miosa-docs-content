---
slug: "faq/getting-started"
title: "入门"
description: "从哪里开始、如何进行身份验证、安装 CLI、获取 API 密钥、将 ID 附加到沙箱以及选择区域。"
section: "Help"
order: 16001
---

关于 MIOSA 第一步的简短回答。

打开类别，或使用列表上方的全部展开和全部折叠。
每个问题都有自己的链接（右侧的图标），因此您可以直接分享答案。

## 我从哪里开始？

[快速入门](/docs/quickstart) 建议首先**轨道 1：沙盒部署**。
您的第一个成功是在生产 URL MIOSA 返回的站点上运行，并进行端到端验证。
完整的演练是[构建和部署 Next.js 应用程序](/docs/learn/first-production-app)。

## 我如何进行身份验证？

使用 `Authorization: Bearer` 标头中的 MIOSA API 密钥，或使用 `POST /auth/login` 中的 JWT。
请参阅[身份验证](/docs/authentication)和[API密钥](/docs/platform/api-keys)。

## 如何安装 CLI 并登录？

从 npm、Homebrew、安装脚本或发布存档安装 `miosa` CLI (v2.2)。
然后运行 ​​`miosa login` 来查看设备代码流，并运行 `miosa whoami` 来确认活动组织和工作区。
请参阅[安装 CLI](/docs/cli/install)。

```bash
miosa login     # device-code flow
miosa whoami    # confirm the active organization and workspace
```

## 如何获取 API 密钥？

在 **工作区 → 设置 → API 密钥** 下创建一个。
明文显示一次，仅存储哈希值。
主要系列包括 `msk_u_`（用户）、`msk_p_`（租户平台）和 `msk_a_`（操作员管理员，非自助服务）。
用户密钥的默认速率为每分钟 600 次请求，平台密钥的默认速率为每分钟 30,000 次；每 90 天轮换一次。
请参阅[API 密钥](/docs/platform/api-keys)。

## 有测试模式吗？

不。
没有专用的测试模式键：`msk_u_*` 和 `msk_p_*` 键击中计费生产基板。
使用沙箱或单独的工作区进行实验。

## 我可以将哪些 ID 附加到沙箱？

自由格式 `metadata`，加上可查询的 `external_user_id`、`external_workspace_id` 和 `external_project_id`。
请参阅[元数据和标签](/docs/develop/sandboxes/metadata)。

## 存在哪些区域，沙箱可以在它们之间移动吗？

`us-west`（默认）、`us-east` 和 `us-mia`，通常都可用。
资源在其生命周期内都驻留在一个区域中，因此请为每个区域创建一个资源。
欧盟和亚太地区已列入路线图。
请参阅[区域](/docs/regions)。
