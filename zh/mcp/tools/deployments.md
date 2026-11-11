---
slug: "mcp/tools/deployments"
title: "部署工具"
description: "托管 MIOSA MCP 部署工具：部署、Docker 主机、版本、环境和日志。"
section: "CLI, SDKs & MCP"
order: 11027
---

该系列中的每个工具都与托管服务器注册的完全相同。论点
是工具自己的参数名称；仍然列出了可选的，因为
描述说明了它们改变了什么。

|工具|论点|描述 |
|---|---|---|
| `deployment_create` | `name`、`repo_url`、`branch`、`build_command`、`run_command`、`deployment_product` |创建新的部署。 |
| `deployment_delete` | `deployment_id` |永久删除部署。 |
| `deployment_env_list` | `deployment_id` |列出部署的环境变量。 |
| `deployment_env_set` | `deployment_id`、`key`、`value` |设置（创建或更新）部署的环境变量。 |
| `deployment_get` | `deployment_id` |获取特定部署的详细信息。 |
| `deployment_list` | - |列出租户中的所有部署。 |
| `deployment_logs` | `deployment_id`、`lines`、`since` |获取部署日志。 |
| `deployment_publish` | `deployment_id`, `source` |发布部署的新版本。 |
| `deployment_rollback` | `deployment_id`、`version_id` |将部署回滚到以前的版本。 |
| `deployment_version_list` | `deployment_id` |列出部署的所有版本。 |
| `deployment_version_promote` | `deployment_id`, `version_id` |将特定版本提升为活动部署。 |
| `docker_deploy_create` | `name`、`repo_url`、`branch`、`build_command`、`run_command`、`docker_deploy_template_id` |在工作区的专用 MIOSA 运行时主机上创建由 git 支持的 Docker 主机部署。 |
| `docker_deploy_doctor` | `deployment_id`、`probe`、`probe_path`、`timeout` |端到端验证 Docker 主机部署：产品标记、|
| `docker_deploy_host_ensure` | `workspace_id`、`region`、`size`、`appliance_image`、`metadata` |创建或返回工作区的专用 Docker Host 设备主机。 |
| `docker_deploy_host_get` | `host_id` |获取一台 Docker Host 设备主机。 |
| `docker_deploy_host_list` | `workspace_id` |列出工作区 Docker Host 设备主机。 |
| `docker_deploy_template_get` | `template_id` |获取一个 Docker Host 应用模板。 |
| `docker_deploy_template_list` | `category`、`runtime`、`framework`、`include_preview` |列出生成的站点、应用程序、API、工作人员和组合堆栈的 Docker Host 应用程序模板。 |
