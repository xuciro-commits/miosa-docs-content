---
slug: "guides/agent-operating-guide"
title: "Agent operating guide"
description: "The CLI and release workflow for AI agents using MIOSA."
section: "Agents"
order: 6000
---

Use this page when an AI agent, coding assistant, CI worker, or operator needs to build and release software through MIOSA.
The CLI is the authoritative happy path - prefer its `--json` output over reconstructing lower-level API calls.

## 1. Discover the current inventory

```bash
miosa catalog --json
miosa deploy list --json
miosa deploy get <deployment> --json
```

`catalog` reports the canonical compute products, templates, sizes, and readiness the platform offers right now.
`deploy list` and `deploy get` return the deployments you already own.
Do not guess a resource id from an old transcript when these commands can return the current inventory.

## 2. Develop in a persistent sandbox

```bash
miosa create my-dev-box --template nextjs --wait
miosa console my-dev-box --cwd /workspace
miosa doctor
miosa preview create my-dev-box 3000 --name candidate
```

`create` provisions the sandbox and, with `--wait`, blocks until it is ready.
`console` opens an interactive shell and resumes the same machine on a later run.
`doctor` checks configuration, connectivity and authentication.
Preview publishes a port at a public URL and never promotes production.

## 3. Publish the exact release

**CLI**

```bash
miosa deploy create my-app --from-sandbox my-dev-box --port 3000 --wait --json
miosa deploy releases <deployment>
miosa deploy promote <deployment> <release>
```

**TypeScript**

```typescript
const result = await miosa.deployments.publish(deploymentId, {
  sourceSandboxId: sandbox.id,
  kind: "auto",                       // "static" | "dynamic" | "auto"
  environment: "production",
  outputPath: "/workspace",
  runCommand: "npm start",            // dynamic deployments only
  port: 3000,                         // dynamic only
})

console.log(result.deployment.public_url)
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
```

A candidate is not production merely because its build succeeded.
Promotion targets one exact immutable release, and the production URL stays stable while the active release changes.

## Pick what you need

| You want to | Start here |
|---|---|
| The full CLI workflow | [Agent CLI workflow](/docs/guides/agent-cli-workflow) |
| Interpret status and protect durable resources | [Agent safety and status](/docs/guides/agent-safety) |
| What agents can do | [What MIOSA agents can do](/docs/guides/agent-capabilities) |
| Configure an agent | [Configure an agent](/docs/guides/configure-agent) |
| Understand the deploy objects | [The deployment model](/docs/docker-host/deployment-model) |

## Next

- [Agent CLI workflow](/docs/guides/agent-cli-workflow): Discovery, scope, sandbox development, preview, publish, promote, recovery, and rollback.
- [Agent safety and status](/docs/guides/agent-safety): Composite health signals, durable resources, and capability boundaries.
- [Publishing](/docs/docker-host/publishing): Understand candidate creation, verification, exact promotion, and receipts.
- [Rollback](/docs/docker-host/rollback): Re-point a deployment at an older version with no rebuild.
- [Availability](/docs/availability): Check whether a capability is available, integrated, self-hostable, gated, or planned.
- [What you can build](/docs/use-cases): Explore agent products, creative systems, vertical platforms, and infrastructure products.
- [Application manifest](/docs/develop/templates): Define the sandbox, services, dependencies, readiness, secrets, and data requirements.
