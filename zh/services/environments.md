---
slug: "services/environments"
title: "服务环境"
description: "每个项目都从生产环境开始。将其分叉为暂存环境或预览环境，并在画布上切换环境。"
section: "Services & projects"
order: 1010
---

环境是项目的独立运行副本。
每个项目都从 `production` 环境开始。
你可以添加 `staging`、预览环境或任何其他需要的环境。

每个环境都有自己的服务、变量和发布。
在暂存环境中的更改不会影响生产环境。

## 在画布上切换与分叉

环境名称位于画布顶部栏，紧挨项目名称。
打开后可查看项目的所有环境，并在其间切换。
画布会重新加载为所选环境，地址中会以 `?env=<name>` 携带该环境。

选择 **Fork** 从当前查看的环境创建一个新环境。
为它命名。
MIOSA 会复制服务和变量，然后显示一份摘要，说明复制了什么、跳过了什么。

## 通过 API

```text
GET  /api/v1/projects/:id/environments
POST /api/v1/projects/:id/environments/:env/duplicate   {"name": "staging"}
```

`duplicate` 会返回新环境、它复制的服务和变量，以及跳过的条目。
项目的服务路由接受 `?env=<name>`；不传时使用 `production`。

## 下一步

- [变量](/docs/services/variables)：变量和引用按环境隔离。
- [发布与回滚](/docs/services/releases)：每个环境都有自己的发布。
- Docker Host 为原先的 Docker Host 部署提供了自己的[环境](/docs/docker-host/environments)。
