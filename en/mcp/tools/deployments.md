---
slug: "mcp/tools/deployments"
title: "Deployment tools"
description: "Hosted MIOSA MCP deployment tools: deployments, Docker Host, versions, env, and logs."
section: "CLI, SDKs & MCP"
order: 11027
---

Every tool in this family, exactly as the hosted server registers it. Arguments
are the tool's own parameter names; optional ones are still listed, because the
description says what they change.

| Tool | Arguments | Description |
|---|---|---|
| `deployment_create` | `name`, `repo_url`, `branch`, `build_command`, `run_command`, `deployment_product` | Create a new deployment. |
| `deployment_delete` | `deployment_id` | Delete a deployment permanently. |
| `deployment_env_list` | `deployment_id` | List environment variables for a deployment. |
| `deployment_env_set` | `deployment_id`, `key`, `value` | Set (create or update) an environment variable for a deployment. |
| `deployment_get` | `deployment_id` | Get details of a specific deployment. |
| `deployment_list` | - | List all deployments in the tenant. |
| `deployment_logs` | `deployment_id`, `lines`, `since` | Get logs for a deployment. |
| `deployment_publish` | `deployment_id`, `source` | Publish a new version of a deployment. |
| `deployment_rollback` | `deployment_id`, `version_id` | Rollback a deployment to a previous version. |
| `deployment_version_list` | `deployment_id` | List all versions of a deployment. |
| `deployment_version_promote` | `deployment_id`, `version_id` | Promote a specific version to be the active deployment. |
| `docker_deploy_create` | `name`, `repo_url`, `branch`, `build_command`, `run_command`, `docker_deploy_template_id` | Create a git-backed Docker Host deployment on the workspace's dedicated MIOSA runtime host. |
| `docker_deploy_doctor` | `deployment_id`, `probe`, `probe_path`, `timeout` | Verify a Docker Host deployment end-to-end: product marker, |
| `docker_deploy_host_ensure` | `workspace_id`, `region`, `size`, `appliance_image`, `metadata` | Create or return the dedicated Docker Host appliance host for a workspace. |
| `docker_deploy_host_get` | `host_id` | Get one Docker Host appliance host. |
| `docker_deploy_host_list` | `workspace_id` | List workspace Docker Host appliance hosts. |
| `docker_deploy_template_get` | `template_id` | Get one Docker Host app template. |
| `docker_deploy_template_list` | `category`, `runtime`, `framework`, `include_preview` | List Docker Host app templates for generated sites, apps, APIs, workers, and compose stacks. |
