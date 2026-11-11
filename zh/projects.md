---
slug: "projects"
title: "项目"
description: "在一个工作区内为一个应用、站点或交付物组织资源，同时不让项目本身成为运行时。"
section: "Services & projects"
order: 1000
---

**项目**是一个工作区内持久的产品边界。
它为一个应用、站点、工作流或客户交付物组织资源，而自身从不运行代码。
属于该产品的每个资源都带有项目的 `project_id`，因此所有权、部署历史、域名、成员和用量都会归集到同一个边界。

一个项目始终只属于一个工作区，而一个工作区属于一个组织。
工作区是必填项：创建项目时未指定工作区会失败并返回 `WORKSPACE_REQUIRED`。

## 1. 创建项目

```bash
miosa project create "Lead Magnet" \
  --workspace-id "$MIOSA_WORKSPACE_ID" \
  --description "Customer lead capture app"

miosa project list
miosa project get lead-magnet
```

## 2. 添加资源

```bash
# Create a sandbox in the workspace, then publish its output.
miosa create lead-magnet-dev --workspace acme-corp --wait
miosa deploy create --from-sandbox lead-magnet-dev --wait
```

## 3. 部署

```bash
miosa deploy create --git https://github.com/me/lead-magnet --branch main --wait
```

发布会在该部署下创建一个不可变版本，并输出 URL。
请显示 API 返回的 URL，不要用 slug 自行拼接。

## 按需选择

| 你想要 | 页面 |
|---|---|
| 查看项目记录、字段以及其中包含的内容 | [项目资源](/docs/projects/resources) |
| 执行完整的创建、配置、部署和停用流程 | [项目生命周期](/docs/projects/lifecycle) |
| 管理成员、权限和计费归属 | [成员与用量](/docs/projects/members-and-usage) |

## 下一步

- [项目资源](/docs/projects/resources)：项目记录、挂接到项目的对象，以及它如何映射到你自己的平台。
- [项目生命周期](/docs/projects/lifecycle)：创建、添加资源、配置环境和密钥、部署以及停用。
- [成员与用量](/docs/projects/members-and-usage)：成员、权限、用量归属以及常见错误。

## 相关内容

- [项目 API](/docs/api-reference/projects)：创建、解析、更新和检查项目。
- [发布](/docs/docker-host/publishing)：从可变源转为不可变候选版本。
- [域名](/docs/docker-host/domains)：配置预览、生成的应用 URL 和精确主机名。
- [产品模型](/docs/concepts)：组织、工作区和项目三层之间如何关联。

组织、工作区和项目构成一个层级：[打开产品模型 →](/docs/concepts)。
