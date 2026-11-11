---
slug: "docker-host/docker-deploy"
title: "Docker Host"
description: "Run many small apps on one always-on workspace host while MIOSA owns routing, domains, and attribution."
section: "Services & projects"
order: 1023
---

Docker Host is MIOSA's dedicated workspace host model for customers that need many small apps, funnels, lead magnets, APIs, or client sites without creating a new production VM for every deployment.
It is the recommended production path for generated apps that should run 24/7 inside a workspace.
MIOSA keeps routing, domains, and attribution as the source of truth, and the host supplies the execution capacity.

## 1. Ensure the workspace host

```bash
miosa api POST /docker-deploy/hosts/ensure -d '{"workspace_id": "<workspace-id>"}'
miosa api GET /docker-deploy/hosts?workspace_id=<workspace-id>
```

The host moves from `provisioning` to `bootstrapping` to `active`. Publishes wait until it is active.

## 2. Publish an app

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

To publish a new version without changing the deployment URL, reuse the same `--name`.

## 3. Prove it

```bash
miosa deploy get <deployment-id> --json
```

A Docker Host deployment is live only when the host is active, the app target is running, the container route is reachable, and a probe to `public_url` succeeds.

## Pick what you need

| You want to | Start here |
|---|---|
| Understand host lifecycle and proof | [Docker Hosts](/docs/docker-host/hosts) |
| Start from a curated app shape | [Docker Host templates](/docs/docker-host/templates) |
| Standard static or dynamic publish | [Deploy overview](/docs/docker-host/overview) |
| Attach a managed or custom domain | [Deployment Domains](/docs/docker-host/domains) |
| Compare standard and Docker Host targets | [The deployment model](/docs/docker-host/deployment-model) |

## Next

- [Docker Hosts](/docs/docker-host/hosts): The always-on workspace host: lifecycle states, publishing flow, proof, and recovery.
- [Docker Host templates](/docs/docker-host/templates): Curated app shapes with Dockerfile, Compose, command, and port defaults.
- [Publishing](/docs/docker-host/publishing): Freeze a sandbox into an immutable deployment version.
- [Deployment domains](/docs/docker-host/domains): Attach managed and custom domains to deployments.
- [White-label setup](/docs/platform/white-label): Configure organization branding and domain behavior.
- [External attribution](/docs/platform/attribution): Map MIOSA resources to your users, workspaces, and projects.
