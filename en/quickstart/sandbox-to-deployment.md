---
slug: "quickstart/sandbox-to-deployment"
title: "Sandbox to deployment"
description: "The full first walkthrough: create a sandbox, write code, preview it live, publish to production, and prove the deployment is running."
section: "Get started"
order: 2
---

Start with **Track 1: Sandbox to deployment**.
The recommended first walkthrough.
Your first success is a small site running at the production URL returned by MIOSA, with its deployment verified.
Complete that path before adding agent orchestration or another runtime.

> **Building a white-label product?** If your goal is to ship a SaaS where your customers never see the MIOSA brand, start with the [White-Label Integrator Guide](/docs/integrations) instead. It covers per-user provisioning, custom domains, iframe embeds, webhooks, and usage billing from scratch.

## Prerequisites

### Account and workspace

Create a MIOSA account at [miosa.ai](https://miosa.ai) and a workspace.

### API key

Create a key in Settings → API Keys for the intended workspace.
See [API keys](/docs/platform/api-keys) for key types and scopes.
Export your key in the terminal where you will run the examples:

```bash
export MIOSA_API_KEY="msk_u_..."
```

### Install tooling for your language

**TypeScript**

```bash
npm install @miosa/cli @miosa/sdk
# or: pnpm add @miosa/cli @miosa/sdk
```

**Python**

```bash
pip install --upgrade miosa
```

**Elixir**

```elixir
# mix.exs  -  published on Hex.pm: https://hex.pm/packages/miosa
{:miosa, "~> 1.1"}
```

Then run `mix deps.get`.

**MCP (Claude Code / Cursor / Windsurf)**

**Zero-install option  -  just a URL + API key:**

```json
{
  "mcpServers": {
    "miosa": {
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": {
        "Authorization": "Bearer msk_u_..."
      }
    }
  }
}
```

Add this to `~/.claude/mcp.json` (Claude Code), Cursor's MCP settings, or Windsurf's `mcp_config.json`. No pip, no npm  -  the current capability-scoped tool catalog appears immediately after a client restart. This is the easiest way to get started.

**Local server (optional):**

```bash
pip install git+https://github.com/Miosa-osa/miosa-mcp.git
```

Or via the TypeScript CLI:

```bash
npx @miosa/cli mcp serve
```

> **Caution: Never put msk_* in a browser**
>
> Workspace API keys are server-only. For browser-side preview embedding, your backend mints a short-lived share token. See [Browser Tokens](/docs/platform/browser-tokens).

## Step 1  -  Create a sandbox.

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  workspaceSlug: "default", // the workspace it belongs to; every create names one
})
console.log(sandbox.id, sandbox.state)
// sbx_a1b2c3d4e5  running
```

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sandbox = miosa.sandboxes.create(template_id="miosa-sandbox", workspace_slug="default")
print(sandbox.id, sandbox.state)
# sbx_a1b2c3d4e5  running
```

## Step 2  -  Write a small static site into `/workspace`.

**TypeScript**

```typescript
await sandbox.files.write(
  "/workspace/index.html",
  `<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>`
)
```

**Python**

```python
sandbox.files.write(
    "/workspace/index.html",
    """<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>""",
)
```

## Step 3  -  Start a dev server so you can preview it.

**TypeScript**

```typescript
// Start the dev server. `exec.stream` keeps the process running and yields its output.
const events = sandbox.exec.stream("python3 -m http.server 3000 --bind 0.0.0.0", { cwd: "/workspace" })
for await (const event of events) console.log(event)
```

**Python**

```python
# Start the dev server in the background.
server = sandbox.commands.run(
    "python3 -m http.server 3000 --bind 0.0.0.0",
    cwd="/workspace",
    background=True,
)
```

## Step 4  -  Create a preview and open it.

**TypeScript**

```typescript
const preview = await sandbox.previews.create(3000)
console.log(preview.url)
// https://3000-abc12345.sandbox.miosa.ai
```

**Python**

```python
preview = sandbox.previews.create(port=3000)
print(preview.url)
# https://5173-abc12345.sandbox.miosa.ai
```

   Open the URL in your browser. You see your `index.html` served live from inside the sandbox.

## Step 5  -  Publish to production.

**CLI**

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name my-app \
  --dir /workspace \
  --type docker_deploy \
  --wait \
  --json
```

Use the `deployment.id` and `deployment.public_url` returned by MIOSA.

**TypeScript**

```typescript
const result = await sandbox.deploy({
  name: "my-app",
  outputPath: "/workspace",
})
// Read the production URL from the response; do not construct it.
console.log(result)
```

**Python**

```python
result = sandbox.deploy(
    name="my-app",
    output_path="/workspace",
)
# Read the production URL from the response; do not construct it.
print(result)
```

   MIOSA freezes a source snapshot, runs a Builder to package the files, stores an immutable Release, and routes the Deployment URL to it. The sandbox remains alive and editable.

## Step 6  -  Prove the deployment.
The production URL is the URL returned by MIOSA.
Do not construct it from the slug.
Use deployment proof when you need an operator or CI check that the app is live.

```bash
miosa deploy get <deployment-id> --json
```

For Docker Host deployments, proof checks the deployment row, Docker Host host link, host health, durable app target, container route, and public URL probe.

## Step 7  -  Make a change and republish.
Edit `/workspace/index.html` again and publish to the existing deployment:

```bash
miosa deploy create --from-sandbox <sandbox-id> \
  --name <deployment> \
  --dir /workspace \
  --type docker_deploy \
  --wait \
  --json
```

MIOSA creates a new immutable Version and promotes it only after verification succeeds.
The previous healthy version remains available for rollback.

## Step 8  -  (Optional) Attach a custom domain.
   See [Domains](/docs/docker-host/domains) for the DNS setup. Short version: add a `CNAME` from your domain to the generated host, hit the verify endpoint, MIOSA issues a TLS certificate.

## Step 9  -  Clean up.

**TypeScript**

```typescript
await sandbox.destroy()
```

**Python**

```python
sandbox.destroy()
```

   The sandbox is gone. The published deployment and its domain are unaffected  -  the Release is immutable and lives in object storage independently of the sandbox.

### What you just did

| Step | What happened |
|------|--------------|
| `sandboxes.create` | Created an isolated sandbox on the sandbox image |
| `files.write` | Wrote a file into `/workspace` inside the running VM |
| `exec` | Ran a command inside the VM over the exec API |
| `previews.create` | Opened a tunnel from a public hostname to port 3000 inside the VM |
| `sandbox publish` | Froze source, ran Builder, stored Release, created Version, and made it live |
| `deploy prove` | Verified that MIOSA can prove the deployment route and runtime state |
| `sandbox.destroy` | Terminated the VM; deployment is unaffected |

## What's next

- **Add a real backend.** Move from static to dynamic deploys with [Publishing](/docs/docker-host/publishing) and a managed [Postgres database](/docs/data/postgres).
- **Authenticate end users.** [Auth-as-a-service](/docs/data/auth) adds a full JWT signup/login flow with one environment variable.
- **Roll back a bad deploy.** [Rollback](/docs/docker-host/rollback) repoints the domain at an older version in seconds.
- **Understand every primitive.** [Concepts](/docs/concepts) is the canonical architecture reference.
- **Build your own product on MIOSA.** The [Platform / White-label overview](/docs/platform/overview) covers preview tokens, share URLs, tenant branding, per-user quotas, webhook subscriptions, Docker Host domains, and the audit log.
- **Drop in UI components.** `@miosa/react`, `@miosa/vue`, and `@miosa/svelte` provide `MiosaPreview`, `MiosaTerminal`, `MiosaFileTree`, and `MiosaUsage`. See [UI Components](/docs/ui-components).
- **See a full white-label example.** The [reference integration](https://github.com/Miosa-osa/miosa/tree/main/examples) shows a complete platform embedding.
- **Extend Claude Code with MIOSA skills.** 5 pre-built skills at [github.com/Miosa-osa/miosa-skills](https://github.com/Miosa-osa/miosa-skills) give Claude Code sandbox, computer, and deploy primitives. See [Skills](/docs/guides/skills).
- **API wire format.** [API Reference](/docs/api-reference) documents every endpoint.
