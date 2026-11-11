---
slug: "mcp/tools"
title: "工具参考"
description: "所有 MIOSA MCP 工具，按系列分组。此概述涵盖工作区、计费和 API 密钥；工具系列链接到它们自己的页面。"
section: "CLI, SDKs & MCP"
order: 11024
---

托管 MIOSA MCP 服务器的每个工具 (`https://api.miosa.ai/api/v1/mcp`)
登记册，按家庭分组。工具名称准确：计算机工具带有前缀
`computer_`，沙箱工具`sandbox_`。所有计算机工具都需要一个可选的
`computer_id` 并回退到活动计算机。

- [沙盒工具](/docs/mcp/tools/sandboxes)：沙盒、执行和文件、预览、部署、快照和模板。
- [计算机工具](/docs/mcp/tools/computers)：生命周期、文件、检查点、环境、域、屏幕和指针、键盘和剪贴板、窗口。
- [部署工具](/docs/mcp/tools/deployments)：部署、Docker 主机、版本、环境和日志。
- [数据工具](/docs/mcp/tools/data)：数据库和对象存储。
- [平台工具](/docs/mcp/tools/platform)：计算机服务、cron、卷、webhooks、函数和区域。

> **注意**
>
> 如果控制平面操作权限未授予该功能，则受控工具将无法关闭。请参阅[操作权限](/docs/mcp#action-authority)。

## 组织和工作空间

|工具|论点|描述 |
|---|---|---|
| `organization_current` | - |获取当前用于 MCP 请求的组织。 |
| `organization_list` | - |列出经过身份验证的用户可用的组织。 |
| `organization_switch` | `organization` |按 ID 或 slug 为每个后续 MCP 请求选择组织。 |
| `workspace_create` | `name`、`description` |创建一个新的工作区。 |
| `workspace_get` | `workspace_id` |获取特定工作区的详细信息。 |
| `workspace_list` | - |列出租户中的所有工作区。 |
| `workspace_stats` | `workspace_id` |获取工作区（计算机、沙箱、数据库等）的资源统计信息。 |
| `workspace_update` | `workspace_id`、`name`、`description` |更新工作区的名称或描述。 |
| `workspace_usage` | `workspace_id`, `period` |获取工作区的使用数据（计算时间、存储、带宽）。 |

## 计费和积分

|工具|论点|描述 |
|---|---|---|
| `billing_plan` | - |获取租户当前的计费计划详细信息。 |
| `billing_usage` | - |获取租户当前的计费周期使用情况。 |

## API 密钥

|工具|论点|描述 |
|---|---|---|
| `api_key_create` | `name`、`scopes`、`items` |创建新的 API 密钥。 |
| `api_key_delete` | `key_id` |撤销和删除 API 密钥。 |
| `api_key_list` | - |列出租户的所有 API 密钥。 |

## 其他所有工具

上面的系列适合此页面。其余的都在
家庭页面：[沙箱](/docs/mcp/tools/sandboxes)，
[计算机](/docs/mcp/tools/computers),
[部署](/docs/mcp/tools/deployments),
[数据](/docs/mcp/tools/data) 和
[平台](/docs/mcp/tools/platform)。
