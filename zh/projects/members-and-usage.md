---
slug: "projects/members-and-usage"
title: "成员与用量"
description: "项目成员、权限、用量归因、操作检查清单以及常见错误。"
section: "Services & projects"
order: 1003
---

访问权限遵循层级结构，并在项目层级进一步收窄：

- 每个工作区成员都会继承该工作区内所有项目的访问权限。
- **项目成员**记录会将某个用户在单个项目中的角色进一步收窄。角色包括 `owner`、`admin`、`developer` 和 `viewer`，且该用户必须已属于父工作区。
- 项目端点受 `projects:read` 和 `projects:write` 作用域控制。绑定到工作区的 API 密钥会自动固定 `workspace_id`，因此无需在请求体中重复填写。
- 成员管理接口为 `GET` 和 `POST /api/v1/projects/{id}/members`，以及 `PATCH` 和 `DELETE /api/v1/projects/{id}/members/{user_id}`。

每条计量记录都通过 `project_id` 归因用量。
项目汇总可通过 `GET /api/v1/projects/{id}/usage` 获取；`GET /api/v1/usage/projects` 会按项目或工作区对已结算费用和运行时长进行分组。
未关联项目的用量会单独计入显式的 `unattributed` 分类，而不会并入某个具名项目，因此总量有缺口时可以直接看出来。
您的 `external_project_id` 会一直传递到用量统计、审计记录和列表筛选中。

## 操作检查清单

- 每个资源都准确归属到一个组织和一个工作区。
- 每个产品资源都带有预期的 `project_id`。
- 开发在沙箱中进行，而不在线上部署中进行。
- 生产变更只能通过发布、验证和提升完成。
- 数据在沙箱替换和应用回滚后仍然保留。
- UI 显示 API 返回的权威 URL。
- 如果计划删除或迁移项目，请确保它不是默认项目。

## 常见错误

| 症状 | 可能原因 | 修正方式 |
|---|---|---|
| 资源显示在错误的客户下 | 工作区或项目是根据过时的 UI 状态推断的 | 显式发送规范 ID |
| 新应用覆盖了旧应用的历史记录 | 两个产品共享同一个项目和部署 | 创建独立的项目和部署 |
| 域名变更不影响沙箱预览 | 更改的是生产主机名 | 单独配置项目预览域名 |
| 删除沙箱似乎会把应用一并删除 | 开发和生产被当作同一资源 | 移除开发状态前先发布到部署 |
| 删除返回 `409 PROJECT_HAS_RESOURCES` | 沙箱、部署或数据仍带有该 `project_id` | 先迁移或删除列出的资源，再执行删除 |
| 删除或迁移返回 `409 DEFAULT_PROJECT_PROTECTED` | 目标是工作区的默认项目 | 创建具名项目并将资源移入其中 |

## 下一步

- [项目 API](/docs/api-reference/projects)：创建、解析、更新和检查项目。
- [工作区](/docs/api-reference/workspaces)：项目之上的团队、客户或业务边界。
- [用量与计费](/docs/platform/usage-and-billing)：在单一 MIOSA 账单上按客户统计用量和分摊费用。
