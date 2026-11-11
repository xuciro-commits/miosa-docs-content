---
slug: "api-reference/docker-deploy"
title: "API Reference / Docker Host"
description: "REST API for Docker Host templates, workspace hosts, and sandbox-to-container publishing."
section: "API reference"
order: 12039
---

Docker Host is the dedicated workspace host for customers who need many containerized apps on one always-on MIOSA-managed runtime.

Use the regular Deployments API for the deployment object.
Use the Docker Host API to choose templates and provision the workspace host.

> **Tip: Routing model**
>
> Docker Host apps still return normal MIOSA deployment URLs and custom-domain records.
> The workspace host runs the app containers while MIOSA owns DNS, TLS, routing, health checks, and deployment records.

## Endpoints

```http
GET  /api/v1/docker-deploy/templates
GET  /api/v1/docker-deploy/templates/:id
GET  /api/v1/docker-deploy/hosts?workspace_id=:workspace_id
POST /api/v1/docker-deploy/hosts/ensure
GET  /api/v1/docker-deploy/hosts/:id

POST /api/v1/sandboxes/:id/publish
POST /api/v1/deployments/:id/publish
```

All endpoints require `Authorization: Bearer <msk_*>`. Mutations should include an `Idempotency-Key` header.

## List templates

```http
GET /api/v1/docker-deploy/templates
```

Returns curated container deployment templates such as Next.js app, Next.js static site, React/Vite, static HTML, Python API, Node API, and compose-backed app patterns.

Response:

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

## Get template

```http
GET /api/v1/docker-deploy/templates/:id
```

Use this when your UI needs the template defaults before creating or publishing a deployment.

## List workspace hosts

```http
GET /api/v1/docker-deploy/hosts?workspace_id=550e8400-e29b-41d4-a716-446655440000
```

Returns Docker Host hosts visible to the caller.

Response:

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

## Ensure workspace host

```http
POST /api/v1/docker-deploy/hosts/ensure
```

Creates or returns the dedicated Docker Host host for a workspace.
One host can run many deployments for that workspace.

Body:

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

Response:

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

## Get host

```http
GET /api/v1/docker-deploy/hosts/:id
```

Returns the latest status for one Docker Host host.
A deployment is not ready until the host is active and the deployed app health check passes through that host.

## Publish from sandbox

```http
POST /api/v1/sandboxes/:id/publish
```

Body:

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

MIOSA packages the sandbox, ensures the workspace Docker Host host exists, installs the release, waits for the app route health check, then returns the deployment and public URL.

## Publish existing deployment

```http
POST /api/v1/deployments/:id/publish
```

Use the same Docker Host fields when publishing a new version of an existing deployment:

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

## See also

- [Docker Host guide](/docs/docker-host/docker-deploy) - end-to-end setup and operating model
- [Deployments](/docs/api-reference/deployments): deployment object, publish, rollback, delete
- [Custom Domains](/docs/api-reference/custom-domains): attach exact domains to deployments
- [Workspaces](/docs/api-reference/workspaces): workspace ownership and attribution
