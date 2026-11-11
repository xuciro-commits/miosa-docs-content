---
slug: "docker-host/overview"
title: "Docker Host"
description: "Your own always-on host that runs apps published from a sandbox, with immutable versions, health checks and rollback. Formerly App Engine."
section: "Services & projects"
order: 1020
---

Docker Host (formerly App Engine) is a box of your own that runs the apps you publish from a sandbox.
For running a project's web services, workers, schedules and static sites, use [Services](/docs/services/overview) instead.

## Docker Host or Services?

| You want | Use |
|---|---|
| A web service, worker, schedule or static site in a project, with releases, variables, domains and rollback | [Services](/docs/services/overview) |
| An app published from a sandbox onto a host you run, with immutable versions | Docker Host (this section) |

Nothing here is removed.
The pages below describe the Docker Host deployment model.

Deploying on MIOSA means turning a Sandbox into a stable production app, site, or API backed by immutable versions, health checks, and a public URL returned by the API.
Once published, the deployment URL stays stable.
New publishes change which version it points at.

## 1. Publish from a sandbox

**CLI**

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name <deployment> \
  --type docker_deploy \
  --wait \
  --json
```

Use the same `--name` on every publish so MIOSA creates a new version behind the same deployment ID and URL.

**TypeScript**

```typescript
const result = await miosa.deployments.publish(deploymentId, {
  sourceSandboxId: sandbox.id,
  kind: "auto",                       // "static" | "dynamic" | "auto"
  environment: "production",          // "production" | "staging" | etc.
  outputPath: "/workspace",
  runCommand: "npm start",            // dynamic deployments only
  port: 3000,                         // dynamic only
})

console.log(result.deployment.public_url)
console.log(result.version.kind, result.version.state)
```

**Python**

```python
result = miosa.deployments.publish(
    deployment_id,
    source_sandbox_id=sandbox_id,
    kind="auto",              # "static" | "dynamic" | "auto"
    environment="production",
    output_path="/workspace",
    run_command="npm start",         # dynamic only
    port=3000,                       # dynamic only
)

print(result.deployment.public_url)
print(result.version.kind, result.version.state)
```

## 2. Promote or roll back

```bash
miosa deploy releases <deployment>
miosa deploy promote <deployment> <release>
miosa deploy rollback <deployment> --to <release-id> --yes --json
```

Promotion and rollback each target one exact immutable release.

## 3. Prove it is live

```bash
miosa deploy get <deployment> --json
miosa deploy requests <deployment>
```

A deployment object is not proof by itself.
The public URL must route to the active release, and for Docker Host also to the workspace host and a running container.

## Pick what you need

| You want to | Start here |
|---|---|
| Serve a static site, SPA, or docs | [Publishing](/docs/docker-host/publishing) |
| Run an API, SSR, or WebSocket server | [Dynamic Runtimes](/docs/docker-host/runtime-instances) |
| Run many small apps on one host | [Docker Host](/docs/docker-host/docker-deploy) |
| Roll back to an older version | [Rollback](/docs/docker-host/rollback) |
| Attach a custom domain | [Deployment Domains](/docs/docker-host/domains) |
| Understand the objects behind a publish | [The deployment model](/docs/docker-host/deployment-model) |

## Next

- [Publishing](/docs/docker-host/publishing): The sandbox-to-version pipeline: source snapshots, build modes, and promotion.
- [Versions](/docs/docker-host/versions): Immutable history. List, inspect, promote, and archive versions.
- [Releases](/docs/docker-host/releases): The artifacts: static files and dynamic server artifacts.
- [Rollback](/docs/docker-host/rollback): Re-point a deployment at an older version in seconds, with no rebuild.
- [Domains](/docs/docker-host/domains): Managed and custom domains. DNS verification, TLS, and route caching.
- [Dynamic Runtimes](/docs/docker-host/runtime-instances): Dynamic deployments  -  production runtimes, health checks, and reconciler behavior.
- [The deployment model](/docs/docker-host/deployment-model): Pipeline, static vs dynamic, deployments, versions, and the proof contract.
