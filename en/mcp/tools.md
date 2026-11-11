---
slug: "mcp/tools"
title: "Tools reference"
description: "All MIOSA MCP tools, grouped by family. This overview covers workspaces, billing, and API keys; the tool families link to their own pages."
section: "CLI, SDKs & MCP"
order: 11024
---

Every tool the hosted MIOSA MCP server (`https://api.miosa.ai/api/v1/mcp`)
registers, grouped by family. Tool names are exact: computer tools are prefixed
`computer_`, sandbox tools `sandbox_`. All computer tools take an optional
`computer_id` and fall back to the active computer.

- [Sandbox tools](/docs/mcp/tools/sandboxes): Sandboxes, exec and files, previews, deploy, snapshots and templates.
- [Computer tools](/docs/mcp/tools/computers): Lifecycle, files, checkpoints, env, domains, screen and pointer, keyboard and clipboard, windows.
- [Deployment tools](/docs/mcp/tools/deployments): Deployments, Docker Host, versions, env, and logs.
- [Data tools](/docs/mcp/tools/data): Databases and object storage.
- [Platform tools](/docs/mcp/tools/platform): Computer services, cron, volumes, webhooks, functions, and regions.

> **Note**
>
> Governed tools fail closed if the control-plane action authority does not grant the capability. See [Action authority](/docs/mcp#action-authority).

## Organizations and workspaces

| Tool | Arguments | Description |
|---|---|---|
| `organization_current` | - | Get the organization currently used for MCP requests. |
| `organization_list` | - | List organizations available to the authenticated user. |
| `organization_switch` | `organization` | Select an organization by ID or slug for every subsequent MCP request. |
| `workspace_create` | `name`, `description` | Create a new workspace. |
| `workspace_get` | `workspace_id` | Get details of a specific workspace. |
| `workspace_list` | - | List all workspaces in the tenant. |
| `workspace_stats` | `workspace_id` | Get resource statistics for a workspace (computers, sandboxes, databases, etc.). |
| `workspace_update` | `workspace_id`, `name`, `description` | Update a workspace's name or description. |
| `workspace_usage` | `workspace_id`, `period` | Get usage data (compute hours, storage, bandwidth) for a workspace. |

## Billing and credits

| Tool | Arguments | Description |
|---|---|---|
| `billing_plan` | - | Get the current billing plan details for the tenant. |
| `billing_usage` | - | Get current billing period usage for the tenant. |

## API keys

| Tool | Arguments | Description |
|---|---|---|
| `api_key_create` | `name`, `scopes`, `items` | Create a new API key. |
| `api_key_delete` | `key_id` | Revoke and delete an API key. |
| `api_key_list` | - | List all API keys for the tenant. |

## Every other tool

The families above are the ones that fit on this page. The rest are on the
family pages: [sandboxes](/docs/mcp/tools/sandboxes),
[computers](/docs/mcp/tools/computers),
[deployments](/docs/mcp/tools/deployments),
[data](/docs/mcp/tools/data) and
[platform](/docs/mcp/tools/platform).
