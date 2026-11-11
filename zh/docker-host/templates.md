---
slug: "docker-host/templates"
title: "Docker 主机模板"
description: "为 Docker 主机精心设计的应用程序形状 - Dockerfile、Compose、命令、端口、环境和持久性默认值。"
section: "Services & projects"
order: 1022
---

Docker Host 模板是为智能体、SDK 和 UI 精心设计的应用程序形状。它们不会替换您的项目代码。它们在发布沙箱或存储库之前提供 Dockerfile、Compose、命令、端口、环境、持久性和设计上下文默认值。

|模板|最适合 |运行时 |
| -------------------------------------- | ------------------------------------------------- | -------------- |
| Next.js 应用 | SaaS 门户、仪表板、客户端应用程序 |节点|
| Next.js 漏斗/登陆页面 |漏斗、铅磁铁、活动网站|节点|
| Vite React SPA |静态 React 应用程序、计算器、小部件 | nginx |
| Express / 节点 API | webhooks、集成、轻量级 API |节点|
| FastAPI服务 | Python API、AI 辅助服务、数据工具 |蟒蛇 |
|静态 HTML 站点 |文档、一页工具、静态下载 | nginx |
|编写全栈应用程序 |一台主机上的 Web + API + Worker + Postgres + Redis | Docker 组合 |

模板元数据由以下方式返回：

```text
GET /api/v1/docker-deploy/templates
GET /api/v1/docker-deploy/templates/:id
```

创建部署时使用模板 ID 作为元数据，以便部署中心可以显示预期的构建形状并稍后恢复默认值：

```json
{
  "deployment_product": "docker_deploy",
  "metadata": {
    "docker_deploy_template_id": "compose-full-stack"
  }
}
```

## 另请参阅

- [Docker Hosts](/docs/docker-host/hosts)：运行 Docker Host 容器的工作区主机。
- [发布](/docs/docker-host/publishing)：生成发布工件的发布管道。
