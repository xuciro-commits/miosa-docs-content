---
slug: "mcp/tools/platform"
title: "Platform tools"
description: "Hosted MIOSA MCP platform tools: computer services, cron, volumes, webhooks, functions, and regions."
section: "CLI, SDKs & MCP"
order: 11029
---

Every tool in this family, exactly as the hosted server registers it. Arguments
are the tool's own parameter names; optional ones are still listed, because the
description says what they change.

| Tool | Arguments | Description |
|---|---|---|
| `api_key_create` | `name`, `scopes`, `items` | Create a new API key. |
| `api_key_delete` | `key_id` | Revoke and delete an API key. |
| `api_key_list` | - | List all API keys for the tenant. |
| `billing_plan` | - | Get the current billing plan details for the tenant. |
| `billing_usage` | - | Get current billing period usage for the tenant. |
| `cron_create` | `computer_id`, `schedule`, `command`, `name` | Create a new cron job that runs a command on a schedule. |
| `cron_delete` | `cron_id` | Delete a cron job permanently. |
| `cron_executions` | `cron_id` | List recent execution history for a cron job. |
| `cron_get` | `cron_id` | Get details of a specific cron job. |
| `cron_list` | `computer_id` | List all cron jobs in the tenant, optionally filtered by computer. |
| `cron_pause` | `cron_id` | Pause a cron job so it stops running on schedule. |
| `cron_resume` | `cron_id` | Resume a paused cron job. |
| `cron_run_now` | `cron_id` | Trigger an immediate one-off execution of a cron job outside its schedule. |
| `function_create` | `name`, `runtime`, `code` | Create a new serverless function. |
| `function_delete` | `function_id` | Delete a serverless function permanently. |
| `function_invoke` | `function_id`, `payload` | Invoke a serverless function and return its response. |
| `function_list` | - | List all serverless functions in the tenant. |
| `organization_current` | - | Get the organization currently used for MCP requests. |
| `organization_list` | - | List organizations available to the authenticated user. |
| `organization_switch` | `organization` | Select an organization by ID or slug for every subsequent MCP request. |
| `product_template_get` | `template_id` | Get one canonical product template, including size readiness and benchmark lane metadata. |
| `product_template_list` | `product` | List canonical product templates across sandbox, computer, and Docker Host appliance products. |
| `product_template_readiness` | `template_id` | Show size readiness for one canonical product template. |
| `region_list` | - | List available regions and their GPU availability. |
| `run_create` | `instruction`, `command`, `target_kind`, `target_id`, `runner`, `provider` | Create a durable run for an agent or command on a Sandbox or Computer. |
| `run_list` | `target_kind`, `target_id`, `status`, `external_workspace_id`, `external_user_id`, `external_project_id` | List durable runs in the current organization. |
| `settings_compute_pricing` | - | Get compute pricing information for available sizes and GPU types. |
| `settings_get` | - | Get all tenant-level settings. |
| `settings_get_branding` | - | Get tenant branding settings (logo, wallpaper, colours). |
| `settings_update_branding` | `desktop_wallpaper_url`, `logo_url` | Update tenant branding settings. |
| `webhook_create` | `url`, `events`, `items` | Create a new webhook endpoint. |
| `webhook_delete` | `webhook_id` | Delete a webhook. |
| `webhook_list` | - | List all webhooks registered in the tenant. |
| `webhook_test` | `webhook_id` | Send a test event delivery to a webhook endpoint. |
| `workspace_create` | `name`, `description` | Create a new workspace. |
| `workspace_get` | `workspace_id` | Get details of a specific workspace. |
| `workspace_list` | - | List all workspaces in the tenant. |
| `workspace_stats` | `workspace_id` | Get resource statistics for a workspace (computers, sandboxes, databases, etc.). |
| `workspace_update` | `workspace_id`, `name`, `description` | Update a workspace's name or description. |
| `workspace_usage` | `workspace_id`, `period` | Get usage data (compute hours, storage, bandwidth) for a workspace. |
