---
slug: "services/sdk"
title: "服务 SDK"
description: "用于服务、发布、变量、地址和项目的 TypeScript 与 Python 方法。"
section: "Services & projects"
order: 1019
---

两个 SDK 暴露相同的操作。
TypeScript SDK 位于 `miosa.services` 和 `miosa.projects`；Python SDK 同样位于 `miosa.services` 和 `miosa.projects`（同步和异步）。

只要 API 接受类型，就可以传入产品类型：

```typescript
await miosa.services.create(projectId, { name: "API", type: "web" });
```

`web` 和 `private` 都会存储为 `kind: server`；如果您已经直接使用 API，请传入 `kind`。

## 服务

| 方法 | 路由 |
|---|---|
| `list(projectId, { environment })` | `GET /projects/:id/services` |
| `get(serviceId)` | `GET /services/:id` |
| `create(projectId, params)` | `POST /projects/:id/services` |
| `update(serviceId, params)` | `PATCH /services/:id` |
| `delete(serviceId)` | `DELETE /services/:id` |
| `deploy(serviceId, params)` | `POST /services/:id/deployments` |
| `redeploy(serviceId)` | `POST /services/:id/redeploy` |
| `restart(serviceId)` | `POST /services/:id/restart` |
| `start(serviceId)` | `POST /services/:id/start` |
| `stop(serviceId)` | `POST /services/:id/stop` |
| `scale(serviceId, params)` | `POST /services/:id/scale` |
| `releases(serviceId)` | `GET /services/:id/deployments` |
| `release(serviceId, releaseId)` | `GET /services/:id/deployments/:did` |
| `events(serviceId, releaseId)` | `GET /services/:id/deployments/:did/events` |
| `rollback(releaseId)` | `POST /service-deployments/:id/rollback` |
| `redeployRelease(releaseId)` | `POST /service-deployments/:id/redeploy` |
| `cancelRelease(releaseId)` | `POST /service-deployments/:id/cancel` |
| `removeRelease(releaseId)` | `DELETE /service-deployments/:id` |
| `variables(serviceId)` | `GET /services/:id/variables` |
| `setVariables(serviceId, variables)` | `PUT /services/:id/variables` |
| `deleteVariables(serviceId, names)` | `DELETE /services/:id/variables/:name` |
| `effectiveVariables(serviceId)` | `GET /services/:id/variables/effective` |
| `revealVariable(serviceId, name)` | `GET /services/:id/variables/:name/reveal` |
| `enablePublicAddress(serviceId, label?)` | `PUT /services/:id/public-address` |
| `disablePublicAddress(serviceId)` | `DELETE /services/:id/public-address` |
| `connection(serviceId)` | `GET /services/:id/connection` |
| `runs(serviceId)` | `GET /services/:id/runs` |
| `settingCapabilities(projectId)` | `GET /projects/:id/services/setting-capabilities` |
| `createSourceUpload(params)` | `POST /uploads` |

Python 方法名为 snake_case（`set_variables`、`enable_public_address` 等），而 `wait_for_release(service_id, release_id)` 会轮询发布，并在失败时抛出带 `RELEASE_FAILED` 或 `RELEASE_TIMEOUT` 代码的 `MiosaError`。

## 项目

| 方法 | 路由 |
|---|---|
| `list({ workspaceId })` | `GET /projects` |
| `get(projectId)` | `GET /projects/:id` |
| `create(params)` | `POST /projects` |
| `update(projectId, params)` | `PATCH /projects/:id` |
| `delete(projectId)` | `DELETE /projects/:id` |
| `move(projectId, targetWorkspaceId)` | `POST /projects/:id/move` |
| `summary(projectId)` | `GET /projects/:id/summary` |
| `environments(projectId)` | `GET /projects/:id/environments` |
| `createEnvironment(projectId, params)` | `POST /projects/:id/environments` |
| `updateEnvironment(projectId, envId, params)` | `PATCH /projects/:id/environments/:env_id` |
| `duplicateEnvironment(projectId, envId, params)` | `POST /projects/:id/environments/:env_id/duplicate` |
| `syncEnvironment(projectId, envId, params)` | `POST /projects/:id/environments/:env_id/sync` |
| `deleteEnvironment(projectId, envId)` | `DELETE /projects/:id/environments/:env_id` |
| `sharedVariables(projectId, { environment })` | `GET /projects/:id/shared-variables` |
| `setSharedVariables(projectId, variables)` | `PUT /projects/:id/shared-variables` |
| `deleteSharedVariable(projectId, name)` | `DELETE /projects/:id/shared-variables/:name` |
| `capabilities()` | `GET /project-capabilities` |
| `templates()` | `GET /project-templates` |
| `template(slug)` | `GET /project-templates/:slug` |
| `applyTemplate(projectId, slug, params)` | `POST /projects/:id/templates/:slug/apply` |

## 示例

TypeScript：

```typescript
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const project = await miosa.projects.create({
  name: "Shop",
  workspaceId: process.env.MIOSA_WORKSPACE_ID!,
});

const api = await miosa.services.create(project.id, {
  name: "API",
  type: "web",
  source: { type: "git", ref: "https://github.com/acme/api" },
  settings: { start_command: "node server.js", replicas: 2 },
});

await miosa.services.enablePublicAddress(api.id);
const release = await miosa.services.deploy(api.id, { message: "first deploy" });
console.log(release.number, release.state);
```

Python：

```python
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

project = miosa.projects.create("Shop", workspace_id=os.environ["MIOSA_WORKSPACE_ID"])

api = miosa.services.create(
    project["id"],
    "API",
    type="web",
    source={"type": "git", "ref": "https://github.com/acme/api"},
    settings={"start_command": "node server.js", "replicas": 2},
)

miosa.services.enable_public_address(api["id"])
release = miosa.services.deploy(api["id"], message="first deploy")
live = miosa.services.wait_for_release(api["id"], release["id"])
print(live["state"])
```

## 异步

Python SDK 在 `AsyncMiosa` 下提供相同的接口，每个方法都需要 `await`，并且有异步的 `wait_for_release`。
