---
slug: "api-reference/agent-runtime-profiles"
title: "智能体运行时配置文件"
description: "为沙箱、计算机和自定义智能体会话配置可重用智能体运行时默认值。"
section: "API reference"
order: 12036
---

智能体运行时配置文件定义 MIOSA 应如何为智能体准备智能体运行时
租户、工作区或项目。使用配置文件标准化 OSA、Codex、Claude
跨沙箱和计算机的代码、Pi、Hermes 或自定义运行时，无需
每次运行时传递相同的模型、工具、连接器、策略和环境。

基本路径：`/api/v1/agent-runtime-profiles`

> **注意：配置文件是默认值，而不是提示**
>
> 配置文件配置运行时形状。智能体运行仍然提供提示、目标、
> 超时和特定于任务的元数据。

## 端点

|方法|路径|描述 |
|---|---|---|
| `GET` | `/api/v1/agent-runtime-profiles` |列出租户的资料 |
| `POST` | `/api/v1/agent-runtime-profiles` |创建个人资料 |
| `GET` | `/api/v1/agent-runtime-profiles/{id}` |获取一份个人资料 |
| `PUT` | `/api/v1/agent-runtime-profiles/{id}` |更新个人资料 |
| `DELETE` | `/api/v1/agent-runtime-profiles/{id}` |删除个人资料 |

可以使用 `workspace_id` 或 `project_id` 过滤列表。

## 创建

```http
POST /api/v1/agent-runtime-profiles
Authorization: Bearer msk_u_...
Content-Type: application/json
```

```json
{
  "name": "lumen-claude",
  "runtime": "claude-code",
  "workspace_id": "ws_123",
  "project_id": "proj_123",
  "description": "Default Lumen coding and browser QA runtime",
  "applies_to": {
    "resources": ["sandbox", "computer"]
  },
  "tools": ["filesystem", "shell", "browser", "artifacts"],
  "connectors": ["anthropic/lumen", "refero/managed"],
  "env": {
    "ANTHROPIC_MODEL": "claude-sonnet-4.6"
  },
  "policy": {
    "approval_required_for": ["deploy", "external_write"],
    "max_child_runs": 25
  },
  "metadata": {
    "owner": "platform"
  },
  "is_default": true
}
```

## 字段

|领域|描述 |
|---|---|
| `name` |人类可读的配置文件名称。在相关租户/工作空间/项目范围内是唯一的。 |
| `runtime` | `osa`、`codex`、`claude`、`claude-code`、`pi`、`hermes` 或 `custom` 之一。 |
| `workspace_id` |可选的工作空间范围。 |
| `project_id` |可选项目范围。 |
| `description` |面向操作员的解释。 |
| `applies_to` |资源选择器，通常为 `{ "resources": ["sandbox", "computer"] }`。 |
| `tools` |运行时应该期望的工具 ID，例如 `filesystem`、`shell`、`browser`、`artifacts` 或 `deployments`。 |
| `connectors` |创建目标运行时时要绑定的连接器 UID。 |
| `env` |非秘密运行时默认值。将提供者密钥放在运行时 env/secrets 中，而不是在这里。 |
| `policy` |产品策略，例如批准、最大子运行或外部写入规则。 |
| `metadata` |产品特定标签和 ID。 |
| `is_default` | MIOSA 是否应默认为范围和资源系列解析此配置文件。 |

## 分辨率

智能体运行可以显式传递 `agent_runtime_profile_id`。如果他们不这样做，
MIOSA 按范围和目标系列解析默认值：

```text
explicit run profile
  -> project default
  -> workspace default
  -> tenant default
  -> no profile
```

该配置文件可以在创建或运行沙箱、计算机和
OpenComputer/BYOC 智能体会话。请求级 `env` 值覆盖非秘密值
该单次运行的配置文件环境值。

## 命令行界面

智能体运行时配置文件没有专用的 `miosa` 子命令；驾驶他们
`miosa api` 与 `/agent-runtime-profiles` 端点。

```bash
miosa api GET /agent-runtime-profiles?workspace_id=ws_123

miosa api POST /agent-runtime-profiles -d '{
  "name": "lumen-claude",
  "runtime": "claude-code",
  "workspace_id": "ws_123",
  "project_id": "proj_123",
  "tools": ["filesystem", "shell", "browser", "artifacts"],
  "connectors": ["anthropic/lumen", "refero/managed"],
  "env": {"ANTHROPIC_MODEL": "claude-sonnet-4.6"},
  "policy": {"approval_required_for": ["deploy", "external_write"]},
  "is_default": true
}'

miosa api PUT /agent-runtime-profiles/arp_123 -d '{"runtime": "codex"}'
miosa api DELETE /agent-runtime-profiles/arp_123
```

## 下一步

- [运行时环境](/docs/api-reference/runtime-env)：存储租户、工作区、项目和目标范围的继承环境变量。
- [智能体运行](/docs/api-reference/agent-runs)：将提示分派到沙箱、计算机或自定义运行时。
- [智能体编排](/docs/guides/agent-orchestration)：将运行时配置文件与组、事件、工件和多智能体扇出结合使用。
