---
slug: "faq/white-label"
title: "白标和平台建设者"
description: "白标架构、品牌 URL、最终用户身份验证、向客户计费和品牌推广。"
section: "Help"
order: 16006
---

关于以您自己的品牌为客户运行 MIOSA 的简短回答。

## 白标如何在 MIOSA 上发挥作用？

您的平台映射到一个 MIOSA 组织，每个下游客户都有一个工作区，每个下游客户构建一个项目。
最终用户永远不需要 MIOSA 帐户。
请参阅[白标平台架构](/docs/platform/overview)。

## 我可以使用哪些品牌 URL？

预览域 (`https://{port}-{slug}.sandbox.{preview_domain}`)、部署域 (`https://{slug}.{deployment_domain}`) 和 MIOSA 回退域 (`https://{slug}.{organization_slug}.miosa.app`)。
请参阅[品牌和域名设置](/docs/platform/white-label)。

## 我的最终用户如何进行身份验证？

切勿将 `msk_*` 键放入浏览器中。
您的后端会铸造短期范围的令牌，而最终用户范围的机密和 OAuth 则以 `external_user_id` 为密钥。
请参阅[归属](/docs/platform/attribution)和[白标凭据](/docs/security/white-label)。

## 我如何通过 MIOSA 向我的客户计费？

MIOSA 在一张账单中向组织所有者收取所有使用费用。
通过在 `workspace_id` 或 `external_*` 字段上对使用情况汇总进行分组，向客户退款。
请参阅[使用情况和计费](/docs/platform/usage-and-billing)。

## 我可以为主机和桌面添加品牌吗？

是的。设置 `custom_app_name`、`custom_logo_url`、`support_url`、`desktop_wallpaper_url` 和 `powered_by_visible` 至 `PUT /api/v1/tenant/branding`。
请参阅[品牌和域名设置](/docs/platform/white-label)。
