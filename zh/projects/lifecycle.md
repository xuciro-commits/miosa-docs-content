---
slug: "projects/lifecycle"
title: "项目生命周期"
description: "创建项目、添加资源、配置环境与密钥、部署，然后下线。"
section: "Services & projects"
order: 1001
---

下面五个步骤就是完整路径：创建、添加资源、配置环境与密钥、部署，然后下线。

## 1. 创建项目

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

响应为 `201`，并返回项目记录。
`workspace_slug` 或 `external_workspace_id` 也可以解析工作区；省略 `slug` 时，它会根据 `name` 派生。

## 2. 添加资源

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

沙箱、计算机、部署、数据库、存储、函数、作业和域名都接受 `project_id`。
当您只有自己的标识符时，传入 `external_project_id`，MIOSA 会为您解析或创建项目。
CLI 中的机器命令接受 `--workspace` 而不是项目，因此项目会从工作区默认值解析；当资源必须落到指定项目时，请使用 API 或 SDK。

## 3. 环境与密钥

运行时环境变量按最具体者优先解析：

```text
tenant defaults < workspace defaults < project defaults < resource env < request env
```

项目级变量会作用于项目中的每个资源，无需将值复制到每个资源中。

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

值在静态存储时加密，且绝不会以明文返回。
单个部署的生产配置是分开的：部署拥有自己的环境（`production`、`staging`、`preview`、`development`）及其各自的配置。

## 4. 部署

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

发布会在部署下创建一个不可变版本，并输出 URL。
请显示 API 返回的 URL；绝不要根据 slug 重新拼接它。

## 5. 归档或删除

项目记录没有状态，因此不存在项目归档这一概念。
归档是资源层面的概念：产品的状态是 `active` 或 `archived`，部署环境的状态是 `active`、`inactive` 或 `archived`。
要下线一个项目，请先移动或删除它的资源，然后删除该项目。

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

删除返回 `204`。
如果项目仍拥有资源，则返回 `409 PROJECT_HAS_RESOURCES`，并附带 `details.impact` 文档列出仍阻止删除的内容；对于工作区的默认项目，则返回 `409 DEFAULT_PROJECT_PROTECTED`。
默认项目以及不是工作区管理员的调用者会被拒绝执行移动操作。
