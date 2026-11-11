---
slug: "api-reference/docker-deploy"
title: "API 参考/Docker 主机"
description: "用于 Docker 主机模板、工作区主机和沙箱到容器发布的 REST API。"
section: "API reference"
order: 12039
---

Docker Host 是一款专用工作区主机，适用于需要在一个始终在线的 MIOSA 管理的运行时上运行许多容器化应用程序的客户。

对部署对象使用常规部署 API。
使用 Docker Host API 选择模板并配置工作区主机。

> **提示：路由模型**
>
> Docker Host 应用程序仍然返回正常的 MIOSA 部署 URL 和自定义域记录。
> 工作区主机运行应用程序容器，而 MIOSA 拥有 DNS、TLS、路由、运行状况检查和部署记录。

## 端点

```http
GET  /api/v1/docker-deploy/templates
GET  /api/v1/docker-deploy/templates/:id
GET  /api/v1/docker-deploy/hosts?workspace_id=:workspace_id
POST /api/v1/docker-deploy/hosts/ensure
GET  /api/v1/docker-deploy/hosts/:id

POST /api/v1/sandboxes/:id/publish
POST /api/v1/deployments/:id/publish
```

所有端点都需要 `Authorization: Bearer <msk_*>`。突变应包含 `Idempotency-Key` 标头。

## 列出模板

```http
GET /api/v1/docker-deploy/templates
```

返回精选的容器部署模板，例如 Next.js 应用程序、Next.js 静态站点、React/Vite、静态 HTML、Python API、Node API 和 compose 支持的应用程序模式。

回应：

```json
{
  "data": [
    {
      "id": "nextjs-app",
      "name": "Next.js App",
      "runtime": "node",
      "build_command": "npm install && npm run build",
      "run_command": "npm start",
      "port": 3000,
      "health_check_path": "/"
    }
  ]
}
```

## 获取模板

```http
GET /api/v1/docker-deploy/templates/:id
```

当您的 UI 在创建或发布部署之前需要模板默认值时，请使用此选项。

## 列出工作区主机

```http
GET /api/v1/docker-deploy/hosts?workspace_id=550e8400-e29b-41d4-a716-446655440000
```

返回调用者可见的 Docker Host 主机。

回应：

```json
{
  "data": [
    {
      "id": "78745c76-d5e6-4541-b651-1018f7cc0d89",
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "computer_id": "cmp_...",
      "status": "active",
      "runtime_status": "healthy",
      "runtime_version": "2026.07"
    }
  ]
}
```

## 确保工作区主机

```http
POST /api/v1/docker-deploy/hosts/ensure
```

创建或返回工作区的专用 Docker Host 主机。
一台主机可以为该工作区运行多个部署。

身体：

```json
{
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "region": "us-central",
  "size": "standard",
  "metadata": {
    "customer": "Lumen"
  }
}
```

回应：

```json
{
  "data": {
    "id": "78745c76-d5e6-4541-b651-1018f7cc0d89",
    "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
    "computer_id": "cmp_...",
    "status": "provisioning",
    "runtime_status": "booting"
  }
}
```

## 获取主机

```http
GET /api/v1/docker-deploy/hosts/:id
```

返回一台 Docker Host 主机的最新状态。
在主机处于活动状态并且部署的应用程序运行状况检查通过该主机之前，部署尚未准备就绪。

## 从沙箱发布

```http
POST /api/v1/sandboxes/:id/publish
```

身体：

```json
{
  "name": "Acme Lead Magnet",
  "deployment_type": "docker_deploy",
  "docker_deploy_template_id": "nextjs-app",
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "project_id": "660e8400-e29b-41d4-a716-446655440001",
  "build_command": "npm install && npm run build",
  "run_command": "npm start",
  "port": 3000,
  "health_check_path": "/",
  "external_workspace_id": "acct_123",
  "external_project_id": "lead_magnet_456"
}
```

MIOSA 打包沙箱，确保工作区 Docker Host 主机存在，安装版本，等待应用程序路由健康检查，然后返回部署和公共 URL。

## 发布现有部署

```http
POST /api/v1/deployments/:id/publish
```

发布现有部署的新版本时使用相同的 Docker Host 字段：

```json
{
  "source_sandbox_id": "sbx_...",
  "deployment_type": "docker_deploy",
  "docker_deploy_template_id": "nextjs-app",
  "build_command": "npm install && npm run build",
  "run_command": "npm start",
  "port": 3000,
  "health_check_path": "/"
}
```

## 另请参阅

- [Docker Host 指南](/docs/docker-host/docker-deploy) - 端到端设置和操作模型
- [Deployments](/docs/api-reference/deployments)：部署对象、发布、回滚、删除
- [自定义域](/docs/api-reference/custom-domains)：将确切的域附加到部署
- [工作空间](/docs/api-reference/workspaces)：工作空间所有权和归属
