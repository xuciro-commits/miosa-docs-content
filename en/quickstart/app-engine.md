---
slug: "quickstart/app-engine"
title: "Docker Host"
description: "Run containerized applications on a workspace Docker Host host, then confirm the app is live."
section: "Get started"
order: 4
---

Use Docker Host when a workspace needs many always-on apps, funnels, APIs, or client sites without provisioning a separate production VM for each deployment.
The visible product name is Docker Host.
The CLI and API still use `docker_deploy` as the internal compatibility value.

## Step 1  -  Ensure the workspace Docker Host host.

```bash
miosa api POST /docker-deploy/hosts/ensure -d '{"workspace_id": "<workspace-id>"}'
miosa api GET /docker-deploy/hosts
```

The host must be active before an Docker Host publish can finish.

## Step 2  -  Publish the sandbox to Docker Host.

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name my-app \
  --dir /workspace \
  --run-command "npm run start" \
  --port 3000 \
  --type docker_deploy \
  --wait \
  --wait-timeout 900s \
  --json
```

The response includes the Deployment and its `public_url`.
That URL may be a MIOSA-managed tenant URL, a workspace deployment domain, a tenant deployment domain, or an exact custom domain.
Always show the returned URL.

## Step 3  -  Confirm the app is live.

```bash
miosa deploy get <deployment-id> --json
miosa deploy requests <deployment-id>
```

`deploy get` shows the deployment's status, product and `public_url`.
`deploy requests` shows the recent HTTP requests served, so you can confirm real traffic reaches the app.

## What you just did

| Step | What happened |
|------|--------------|
| `docker-deploy ensure` | Created or returned the workspace Docker Host host |
| `sandbox publish --docker-deploy` | Published the sandbox artifact as a container on that workspace host |
| `docker-deploy doctor` | Proved the container and public URL are real |

> **Note: Read more**
>
> See [Docker Host](/docs/docker-host/docker-deploy) for the full deployment contract, and [Deploy overview](/docs/docker-host/overview) for the rest of the deploy surface.
