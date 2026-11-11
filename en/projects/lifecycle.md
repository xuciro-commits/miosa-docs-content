---
slug: "projects/lifecycle"
title: "Project lifecycle"
description: "Create a project, add resources, configure environments and secrets, deploy, then retire it."
section: "Services & projects"
order: 1001
---

The five steps below are the whole path: create, add resources, configure environments and secrets, deploy, then retire.

## 1. Create a project

**CLI**

```bash
miosa project create "Lead Magnet" \
  --workspace-id "$MIOSA_WORKSPACE_ID" \
  --description "Customer lead capture app"

miosa project list
miosa project get lead-magnet
```

**REST**

```bash
curl -X POST https://api.miosa.ai/api/v1/projects \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workspace_id": "'"$MIOSA_WORKSPACE_ID"'",
    "name": "Lead Magnet",
    "slug": "lead-magnet",
    "external_project_id": "project_789"
  }'
```

The response is `201` with the project record.
`workspace_slug` or `external_workspace_id` also resolve the workspace; `slug` is derived from `name` when you omit it.

## 2. Add resources

**CLI**

```bash
# Create a sandbox in the workspace, then publish its output.
miosa create lead-magnet-dev --workspace acme-corp --wait
miosa deploy create --from-sandbox lead-magnet-dev --wait
```

**TypeScript**

```ts
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY });

// Project CRUD is REST and CLI only; the SDK lists projects per workspace.
const projects = await miosa.workspaces.listProjects(workspaceId);
const project = projects.find((p) => p.slug === "lead-magnet");

const sandbox = await miosa.sandboxes.create({
  projectId: project.id as string,
  projectSlug: "lead-magnet",
});
```

**Python**

```python
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

projects = client.workspaces.list_projects(workspace_id)
project = next(p for p in projects if p["slug"] == "lead-magnet")

sandbox = client.sandboxes.create(
    project_id=project["id"],
    project_slug="lead-magnet",
)
```

**REST**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "project_id": "'"$PROJECT_ID"'", "name": "lead-magnet-dev" }'
```

Sandboxes, computers, deployments, databases, storage, functions, jobs, and domains all accept `project_id`.
When you only have your own identifier, pass `external_project_id` and MIOSA resolves or creates the project for you.
Machine commands in the CLI take `--workspace`, not a project, so the project resolves from the workspace default; use the API or an SDK when a resource must land in a named project.

## 3. Environments and secrets

Runtime environment variables resolve narrowest-wins:

```text
tenant defaults < workspace defaults < project defaults < resource env < request env
```

A project-scoped variable reaches every resource in the project without copying values into each one.

**CLI**

```bash
# CLI environments belong to a workspace or the organization.
echo "$STRIPE_SECRET_KEY" | miosa env set --env staging STRIPE_KEY
```

**TypeScript**

```ts
await miosa.runtimeEnv.set({
  scope: "project",
  projectId: project.id as string,
  name: "STRIPE_SECRET_KEY",
  value: process.env.STRIPE_SECRET_KEY!,
});
```

**Python**

```python
client.runtime_env.set(
    scope="project",
    project_id=project["id"],
    name="STRIPE_SECRET_KEY",
    value=os.environ["STRIPE_SECRET_KEY"],
)
```

**REST**

```bash
curl -X POST https://api.miosa.ai/api/v1/runtime-env \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "scope": "project",
    "project_id": "'"$PROJECT_ID"'",
    "target": "all",
    "name": "STRIPE_SECRET_KEY",
    "value": "sk_live_..."
  }'
```

Values are encrypted at rest and never returned in plaintext.
Production configuration for a single deployment is separate: a Deployment owns its own environments (`production`, `staging`, `preview`, `development`) with their own config.

## 4. Deploy

**CLI**

```bash
miosa deploy create --git https://github.com/me/lead-magnet --branch main --wait
```

**TypeScript**

```ts
// The sandbox already carries the project, so its deployment inherits it.
const deployment = await sandbox.deploy({ name: "lead-magnet" });
```

**Python**

```python
# The sandbox already carries the project, so its deployment inherits it.
deployment = sandbox.deploy(name="lead-magnet")
```

**REST**

```bash
# Create the deployment inside the project, then publish a version.
curl -X POST https://api.miosa.ai/api/v1/deployments \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "name": "lead-magnet", "project_id": "'"$PROJECT_ID"'", "repo_url": "https://github.com/me/lead-magnet", "branch": "main" }'

curl -X POST https://api.miosa.ai/api/v1/deployments/"$DEPLOYMENT_ID"/publish \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Publishing creates an immutable version under the Deployment and prints the URL.
Display the URL the API returns; never rebuild it from a slug.

## 5. Archive or delete

A project row has no state, so there is no project archive.
Archive is a resource concept: a Product is `active` or `archived`, and a deployment environment is `active`, `inactive`, or `archived`.
To retire a project, move or delete its resources first, then delete the project.

**CLI**

```bash
# The CLI creates, lists, and shows projects.
# Moving and deleting a project are REST operations.
miosa project list
miosa project get lead-magnet
```

**REST**

```bash
# Move the project and its children to another workspace.
curl -X POST https://api.miosa.ai/api/v1/projects/"$PROJECT_ID"/move \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "target_workspace_id": "'"$TARGET_WORKSPACE_ID"'" }'

# Delete the project once it owns nothing.
curl -X DELETE https://api.miosa.ai/api/v1/projects/"$PROJECT_ID" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Delete returns `204`.
It returns `409 PROJECT_HAS_RESOURCES` with a `details.impact` document listing what still blocks the delete, and `409 DEFAULT_PROJECT_PROTECTED` for a workspace's Default project.
Move is refused for the Default project and for a caller who is not a workspace admin.
