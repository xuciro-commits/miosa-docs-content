---
slug: "mcp/tools/platform"
title: "平台工具"
description: "托管 MIOSA MCP 平台工具：计算机服务、cron、卷、webhooks、函数和区域。"
section: "CLI, SDKs & MCP"
order: 11029
---

该系列中的每个工具都与托管服务器注册的完全相同。论点
是工具自己的参数名称；仍然列出了可选的，因为
描述说明了它们改变了什么。

|工具|论点|描述 |
|---|---|---|
| `api_key_create` | `name`、`scopes`、`items` |创建新的 API 密钥。 |
| `api_key_delete` | `key_id` |撤销和删除 API 密钥。 |
| `api_key_list` | - |列出租户的所有 API 密钥。 |
| `billing_plan` | - |获取租户当前的计费计划详细信息。 |
| `billing_usage` | - |获取租户当前的计费周期使用情况。 |
| `cron_create` | `computer_id`、`schedule`、`command`、`name` |创建一个按计划运行命令的新 cron 作业。 |
| `cron_delete` | `cron_id` |永久删除 cron 作业。 |
| `cron_executions` | `cron_id` |列出 cron 作业的最近执行历史记录。 |
| `cron_get` | `cron_id` |获取特定 cron 作业的详细信息。 |
| `cron_list` | `computer_id` |列出租户中的所有 cron 作业，可以选择按计算机过滤。 |
| `cron_pause` | `cron_id` |暂停 cron 作业，使其停止按计划运行。 |
| `cron_resume` | `cron_id` |恢复暂停的 cron 作业。 |
| `cron_run_now` | `cron_id` |在计划之外立即触发 cron 作业的一次性执行。 |
| `function_create` | `name`、`runtime`、`code` |创建一个新的无服务器函数。 |
| `function_delete` | `function_id` |永久删除无服务器功能。 |
| `function_invoke` | `function_id`、`payload` |调用无服务器函数并返回其响应。 |
| `function_list` | - |列出租户中的所有无服务器功能。 |
| `organization_current` | - |获取当前用于 MCP 请求的组织。 |
| `organization_list` | - |列出经过身份验证的用户可用的组织。 |
| `organization_switch` | `organization` |按 ID 或 slug 为每个后续 MCP 请求选择组织。 |
| `product_template_get` | `template_id` |获取一份规范的产品模板，包括尺寸准备情况和基准通道元数据。 |
| `product_template_list` | `product` |列出沙箱、计算机和 Docker Host 设备产品的规范产品模板。 |
| `product_template_readiness` | `template_id` |显示一种规范产品模板的尺寸准备情况。 |
| `region_list` | - |列出可用区域及其 GPU 可用性。 |
| `run_create` | `instruction`、`command`、`target_kind`、`target_id`、`runner`、`provider` |在沙盒或计算机上为智能体或命令创建持久运行。 |
| `run_list` | `target_kind`、`target_id`、`status`、`external_workspace_id`、`external_user_id`、`external_project_id` |列出当前组织中的持久运行。 |
| `settings_compute_pricing` | - |获取可用大小和 GPU 类型的计算定价信息。 |
| `settings_get` | - |获取所有租户级别设置。 |
| `settings_get_branding` | - |获取租户品牌设置（徽标、壁纸、颜色）。 |
| `settings_update_branding` | `desktop_wallpaper_url`、`logo_url` |更新租户品牌设置。 |
| `webhook_create` | `url`、`events`、`items` |创建一个新的 Webhook 端点。 |
| `webhook_delete` | `webhook_id` |删除网络钩子。 |
| `webhook_list` | - |列出在租户中注册的所有 Webhook。 |
| `webhook_test` | `webhook_id` |将测试事件发送到 Webhook 端点。 |
| `workspace_create` | `name`、`description` |创建一个新的工作区。 |
| `workspace_get` | `workspace_id` |获取特定工作区的详细信息。 |
| `workspace_list` | - |列出租户中的所有工作区。 |
| `workspace_stats` | `workspace_id` |获取工作区（计算机、沙箱、数据库等）的资源统计信息。 |
| `workspace_update` | `workspace_id`、`name`、`description` |更新工作区的名称或描述。 |
| `workspace_usage` | `workspace_id`、`period` |获取工作区的使用数据（计算时间、存储、带宽）。 |
