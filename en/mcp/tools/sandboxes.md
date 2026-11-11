---
slug: "mcp/tools/sandboxes"
title: "Sandbox tools"
description: "Hosted MIOSA MCP sandbox tools: create, exec, files, previews, deploy, snapshots and templates."
section: "CLI, SDKs & MCP"
order: 11025
---

Every tool in this family, exactly as the hosted server registers it. Arguments
are the tool's own parameter names; optional ones are still listed, because the
description says what they change.

| Tool | Arguments | Description |
|---|---|---|
| `sandbox_create` | `name`, `template_id`, `size`, `cpu_count`, `memory_mb`, `disk_size_mb` | Create a lightweight code sandbox using a canonical named size. |
| `sandbox_deploy` | `sandbox_id`, `name`, `output_path`, `entrypoint`, `port`, `deployment_type` | Deploy sandbox contents to production through MIOSA. |
| `sandbox_deploy_docker` | `sandbox_id`, `name`, `output_path`, `entrypoint`, `port`, `docker_deploy_template_id` | Deploy sandbox contents through the workspace Docker Host runtime. Uses MIOSA native APIs, not an external deploy MCP. |
| `sandbox_destroy` | `sandbox_id` | Destroy a sandbox permanently. |
| `sandbox_exec` | `sandbox_id`, `command`, `cwd`, `timeout` | Run a bash command inside a sandbox and return the output. |
| `sandbox_expose` | `sandbox_id`, `port` | Expose a port on a sandbox and return a publicly accessible preview URL. |
| `sandbox_get` | `sandbox_id` | Get details of a specific sandbox. |
| `sandbox_list` | `state` | List all sandboxes in the tenant. |
| `sandbox_list_files` | `sandbox_id`, `path`, `depth` | List files in a directory inside a sandbox. |
| `sandbox_logs` | `sandbox_id`, `lines` | Get logs from a sandbox. |
| `sandbox_pause` | `sandbox_id` | Pause a running sandbox (suspends it to save compute). |
| `sandbox_python` | `sandbox_id`, `code`, `timeout` | Run Python code inside a sandbox and return stdout, stderr, and exit code. |
| `sandbox_read_file` | `sandbox_id`, `path` | Read a file from inside a sandbox and return its content as text. |
| `sandbox_resume` | `sandbox_id` | Resume a paused sandbox. |
| `sandbox_snapshot_create` | `sandbox_id`, `comment` | Create a snapshot of the current sandbox state. |
| `sandbox_snapshot_list` | `sandbox_id` | List all snapshots for a sandbox. |
| `sandbox_snapshot_restore` | `sandbox_id`, `snapshot_id` | Restore a sandbox from a specific snapshot. |
| `sandbox_template_builds` | `template_id` | List builds for a sandbox template. |
| `sandbox_template_create` | `name`, `build_spec`, `slug`, `description` | Create a custom sandbox template from a build spec. |
| `sandbox_template_get` | `template_id` | Get details of a specific sandbox template. |
| `sandbox_template_list` | - | List tenant-owned sandbox templates for custom sandbox environments. For canonical product/template/size readiness, use product_template_list. |
| `sandbox_upload` | `sandbox_id`, `path`, `content` | Upload binary content into a sandbox. `content` must be |
| `sandbox_write_file` | `sandbox_id`, `path`, `content` | Write text or binary content to a file path inside a sandbox. |
