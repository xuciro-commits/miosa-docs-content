---
slug: "mcp/tools/sandboxes"
title: "沙箱工具"
description: "托管 MIOSA MCP 沙箱工具：创建、执行、文件、预览、部署、快照和模板。"
section: "CLI, SDKs & MCP"
order: 11025
---

该系列中的每个工具都与托管服务器注册的完全相同。论点
是工具自己的参数名称；仍然列出了可选的，因为
描述说明了它们改变了什么。

|工具|论点|描述 |
|---|---|---|
| `sandbox_create` | `name`、`template_id`、`size`、`cpu_count`、`memory_mb`、`disk_size_mb` |使用规范命名大小创建轻量级代码沙箱。 |
| `sandbox_deploy` | `sandbox_id`、`name`、`output_path`、`entrypoint`、`port`、`deployment_type` |通过 MIOSA 将沙箱内容部署到生产环境。 |
| `sandbox_deploy_docker` | `sandbox_id`、`name`、`output_path`、`entrypoint`、`port`、`docker_deploy_template_id` |通过工作区 Docker Host 运行时部署沙箱内容。使用 MIOSA 本机 API，而不是外部部署 MCP。 |
| `sandbox_destroy` | `sandbox_id` |永久销毁沙箱。 |
| `sandbox_exec` | `sandbox_id`、`command`、`cwd`、`timeout` |在沙箱内运行 bash 命令并返回输出。 |
| `sandbox_expose` | `sandbox_id`、`port` |公开沙盒上的端口并返回可公开访问的预览 URL。 |
| `sandbox_get` | `sandbox_id` |获取特定沙箱的详细信息。 |
| `sandbox_list` | `state` |列出租户中的所有沙箱。 |
| `sandbox_list_files` | `sandbox_id`、`path`、`depth` |列出沙箱内目录中的文件。 |
| `sandbox_logs` | `sandbox_id`、`lines` |从沙箱中获取日志。 |
| `sandbox_pause` | `sandbox_id` |暂停正在运行的沙箱（暂停它以节省计算）。 |
| `sandbox_python` | `sandbox_id`、`code`、`timeout` |在沙箱内运行 Python 代码并返回 stdout、stderr 和退出代码。 |
| `sandbox_read_file` | `sandbox_id`、`path` |从沙箱内部读取文件并将其内容作为文本返回。 |
| `sandbox_resume` | `sandbox_id` |恢复暂停的沙箱。 |
| `sandbox_snapshot_create` | `sandbox_id`、`comment` |创建当前沙箱状态的快照。 |
| `sandbox_snapshot_list` | `sandbox_id` |列出沙箱的所有快照。 |
| `sandbox_snapshot_restore` | `sandbox_id`、`snapshot_id` |从特定快照恢复沙箱。 |
| `sandbox_template_builds` | `template_id` |列出沙箱模板的构建。 |
| `sandbox_template_create` | `name`、`build_spec`、`slug`、`description` |根据构建规范创建自定义沙箱模板。 |
| `sandbox_template_get` | `template_id` |获取特定沙箱模板的详细信息。 |
| `sandbox_template_list` | - |列出自定义沙箱环境的租户拥有的沙箱模板。对于规范的产品/模板/尺寸准备情况，请使用product_template_list。 |
| `sandbox_upload` | `sandbox_id`、`path`、`content` |将二进制内容上传到沙箱中。 `content` 必须是 |
| `sandbox_write_file` | `sandbox_id`、`path`、`content` |将文本或二进制内容写入沙箱内的文件路径。 |
