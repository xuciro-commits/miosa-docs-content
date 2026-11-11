---
slug: "api-reference/runtime-env"
title: "运行时环境"
description: "存储沙箱、计算机、智能体和部署的继承运行时环境变量。"
section: "API reference"
order: 12037
---

Runtime Env 存储租户、工作区和环境的继承环境变量
项目范围。值在静态时被加密并且永远不会返回
明文。 MIOSA 将它们具体化为沙箱、计算机、智能体或
根据您选择的目标进行部署。

基本路径：`/api/v1/runtime-env`

> **注意：故意将此用于秘密**
>
> Runtime Env 可以保存提供者密钥，例如 Anthropic、OpenAI、Higgsfield、
> 重新发送、Stripe 或客户 API 令牌。尽可能缩小范围并
> 避免将租户管理员凭据置于广泛的默认值中。

## 端点

|方法|路径|描述 |
|---|---|---|
| `GET` | `/api/v1/runtime-env` |列出继承的环境变量 |
| `POST` | `/api/v1/runtime-env` |创建或更新一个变量 |
| `GET` | `/api/v1/runtime-env/{id}` |显示一个没有明文的变量 |
| `DELETE` | `/api/v1/runtime-env/{id}` |删除一个变量 |

列表支持这些过滤器：

|查询 |价值观 |
|---|---|
| `scope` | `tenant`、`workspace`、`project` |
| `workspace_id` |工作区范围变量的工作区 ID |
| `project_id` |项目范围变量的项目 ID |
| `target` | `all`、`sandbox`、`computer`、`agent`、`deployment` |

## 创建

```http
POST /api/v1/runtime-env
Authorization: Bearer msk_u_...
Content-Type: application/json
```

```json
{
  "scope": "workspace",
  "workspace_id": "ws_123",
  "target": "sandbox",
  "name": "ANTHROPIC_API_KEY",
  "value": "sk-ant-...",
  "metadata": {
    "provider": "anthropic",
    "owner": "lumen"
  }
}
```

环境变量名称必须是大写的 shell 安全名称匹配
`/^[A-Z][A-Z0-9_]*$/`。

## 范围模型

|范围 |必填 ID |用它来 |
|---|---|---|
| `tenant` |无 |组织范围内的默认值。 |
| `workspace` | `workspace_id` |客户/工作空间提供商密钥或工具配置。 |
| `project` | `project_id` |单个应用程序或部署管道。 |

|目标|接收变量 |
|---|---|
| `all` |所有支持的运行时目标。 |
| `sandbox` |沙箱会话和沙箱智能体运行。 |
| `computer` |桌面计算机会话和计算机智能体运行。 |
| `agent` |智能体运行时会话。 |
| `deployment` |生产部署/运行时启动。 |

对可支出的密钥使用更窄的范围。例如，设置
`ANTHROPIC_API_KEY` 在流明的工作空间范围而不是租户范围，如果
该密钥应该只为 Lumen 沙箱和计算机提供动力。

## 命令行界面

Runtime Env 没有专用的 `miosa` 子命令；用 `miosa api` 驱动它
针对 `/runtime-env` 端点。

```bash
miosa api POST /runtime-env -d '{
  "name": "ANTHROPIC_API_KEY",
  "value": "sk-ant-...",
  "scope": "workspace",
  "workspace_id": "ws_123",
  "target": "sandbox"
}'

miosa api POST /runtime-env -d '{
  "name": "HIGGSFIELD_API_KEY",
  "value": "hgf_...",
  "scope": "project",
  "project_id": "proj_123",
  "target": "agent"
}'

miosa api GET /runtime-env?scope=workspace&workspace_id=ws_123
miosa api GET /runtime-env/env_123
miosa api DELETE /runtime-env/env_123
```

## 与配置文件和连接器的关系

运行时环境、智能体运行时配置文件和托管连接器是独立的：

|原始|拥有 |
|---|---|
|运行时环境 |继承的环境变量和加密的提供程序密钥。 |
|智能体运行时简介 |运行时选择、工具、连接器、默认环境和策略。 |
|托管连接器|智能体提供商访问和范围内的令牌交换。 |

使用 Runtime Env 作为直接提供程序环境变量。使用连接器时
MIOSA 应智能体提供商访问或限制出口。使用配置文件进行附加
右侧默认为新的沙箱、计算机和智能体运行。

## 下一步

- [智能体运行时配置文件](/docs/api-reference/agent-runtime-profiles)：标准化每个工作区或项目的运行时、工具、连接器和策略。
- [智能体运行](/docs/api-reference/agent-runs)：使用继承的环境和运行时配置文件解析来调度提示。
- [智能体编排](/docs/guides/agent-orchestration)：将继承的环境、配置文件、组、事件和工件放在一起。
