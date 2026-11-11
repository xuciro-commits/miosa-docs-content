---
slug: "services/sdk"
title: "Services SDKs"
description: "TypeScript and Python methods for services, releases, variables, addresses and projects."
section: "Services & projects"
order: 1019
---

Both SDKs expose the same operations.
The TypeScript SDK lives on `miosa.services` and `miosa.projects`; the Python SDK on `miosa.services` and `miosa.projects` (sync and async).

Product types are accepted everywhere the API takes a kind:

```typescript
await miosa.services.create(projectId, { name: "API", type: "web" });
```

`web` and `private` both store `kind: server`; pass `kind` when you already speak the API.

## Services

| Method | Route |
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

The Python names are snake_case (`set_variables`, `enable_public_address`, ...), and `wait_for_release(service_id, release_id)` polls a release and raises `MiosaError` with code `RELEASE_FAILED` or `RELEASE_TIMEOUT`.

## Projects

| Method | Route |
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

## Examples

TypeScript:

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

Python:

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

## Async

The Python SDK has the same surface under `AsyncMiosa`, with `await` on every method and an async `wait_for_release`.
