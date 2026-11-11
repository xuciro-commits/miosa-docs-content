---
slug: "services/previews"
title: "预览环境"
description: "打开拉取请求，MIOSA 就会将其部署到拥有独立地址的专属环境中，并在拉取请求关闭后将其移除。"
section: "Services & projects"
order: 1011
---

预览环境是项目默认环境的副本，用于运行某个拉取请求。
打开拉取请求后，MIOSA 会创建该环境、部署拉取请求的分支，并在拉取请求上以评论形式给出地址。
关闭或合并拉取请求后，该环境会被删除。

## 会发生什么

| 拉取请求事件 | MIOSA 的操作 |
|---|---|
| 打开或重新打开 | 将默认环境分叉为 `pr-<number>`，部署拉取请求分支上的所有服务，并以评论形式发布地址 |
| 推送新提交 | 重新部署 `pr-<number>` 的服务 |
| 关闭或合并 | 删除 `pr-<number>`，不影响其他任何内容 |

预览中的每个服务器服务都命名为 `<service>-pr-<number>`。
变量会从分叉来源的环境复制。
预览绝不会更改它来源的环境。

## 启用预览

在控制台中打开项目的**设置**，选择**预览**。
将仓库设置为 `owner/name`，开启预览并保存。
页面会显示 webhook URL 和要粘贴到 GitHub 的密钥。

在 GitHub 仓库中添加 webhook：

| 字段 | 值 |
|---|---|
| 负载 URL | 预览页面显示的 URL，`https://api.miosa.ai/api/v1/hooks/git/<project id>` |
| 内容类型 | `application/json` |
| 密钥 | 预览页面中的密钥 |
| 事件 | 仅拉取请求 |

设置的仓库必须与 GitHub 的 `full_name` 完全一致，否则投递会被忽略。
只有签名与密钥匹配时，投递才会被接受。
该密钥为只写属性：它不会再次显示，因此请自行留存副本或设置新密钥。

## 成本控制

| 设置 | 默认值 | 范围 |
|---|---|---|
| `max_open` | 3 | 1 到 20 |
| `ttl_hours` | 72 | 1 到 720 |

已有三个预览打开时，不会创建第四个预览。
无人关闭的拉取请求所对应的预览会在 72 小时后删除。
可以在“预览”中调高或调低这两个值。

## 通过 API

```text
GET    /api/v1/projects/:id/previews
PUT    /api/v1/projects/:id/previews
DELETE /api/v1/projects/:id/previews/:preview_id
```

`PUT` 会替换整行，因此请发送完整对象：`{repository, enabled, max_open, ttl_hours, secret}`。
`secret` 为空或缺失时，保留已存储的密钥。
返回的行只会报告 `has_secret`（绝不返回密钥本身），以及要使用的 `webhook_path`。
这些路由需要 `projects:read` 和 `projects:write` 权限范围。

## 接下来

- [环境](/docs/services/environments)：手动分叉环境。
- [变量](/docs/services/variables)：预览会复制哪些内容。
