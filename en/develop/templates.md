---
slug: "develop/templates"
title: "Templates"
description: "The one sandbox image, the templates that stage content inside it, the desktop, and how to start a Next.js, FastAPI, or static project."
section: "Sandboxes"
order: 2015
---

A **template** is content staged inside the sandbox at create time: files, a scaffold, extra packages, and the command that starts your app.
A template is never a separate image.

Every sandbox boots the same main image, which already has Node, Python, Docker Engine 29.9 with `docker compose` and `docker buildx`, a live browser with CDP on a virtual display, and the code interpreter inside.

## New sandbox

Create a sandbox with no template for code, commands, and artifacts.
Use `template_id` when you want a scaffold staged inside the same image.

**Python**

```python
import miosa

client = miosa.Miosa(api_key="msk_u_...")

sandbox = client.sandboxes.create()
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const miosa = new Miosa({ apiKey: 'msk_u_...' });

const sandbox = await miosa.sandboxes.create();
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Start from a template

Pass `template_id` to stage a scaffold inside the sandbox image and start it.
The template provisions the files, the packages, and the command that runs your app; it is not a separate image.

| `template_id` | What it stages |
|---|---|
| `nextjs` | A Next.js app, started on port 3000. |
| `nextjs-postgres` | A Next.js app with Postgres client tooling on port 3000; bind `DATABASE_URL`. |
| `static-html` | Static HTML, CSS, and JavaScript served from `/workspace` on port 8000. |
| `fastapi-auth` | A FastAPI starter with signup, login, me, and dashboard routes, on port 8000. Its health check does not work yet, so do not gate your own readiness on it. |
| your own | Any custom template your workspace has built. See [Custom templates](/docs/develop/templates#custom-templates). |

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="nextjs",
    auto_start=True,
)
```

**TypeScript**

```typescript
const sandbox = await miosa.sandboxes.create({
  templateId: 'nextjs',
  autoStart: true,
});
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{ "template_id": "nextjs", "auto_start": true }'
```

With `auto_start`, MIOSA starts the template once the sandbox reaches `running` and the response carries the preview URL.
Leave it `false` when you want to write your own files into `/workspace` first, then call [Start a template app](/docs/api-reference/sandboxes/create#start-a-template-app).

### Next.js

**Port 3000.** Next.js SSR deployments need `output: "standalone"` in `next.config.js` so the builder can package the server.
Static exports (`output: "export"`) work without it.

Or do it by hand against a sandbox you created without `auto_start`:

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/template/start \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "install_command": "npm install",
    "start_command": "npm run dev -- --hostname 0.0.0.0 --port 3000",
    "port": 3000
  }'
```

### FastAPI

**Port 8000.** Or do it by hand:

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/template/start \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "install_command": "pip install -r requirements.txt",
    "start_command": "uvicorn main:app --host 0.0.0.0 --port 8000",
    "port": 8000
  }'
```

### Static HTML

**Port 8000.** Plain HTML, CSS, and JavaScript, with no build step.
Or do it by hand:

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/template/start \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "start_command": "python -m http.server 8000 --bind 0.0.0.0",
    "port": 8000
  }'
```

The builder detects a static site (no `package.json`, `Cargo.toml`, or `go.mod`) and packages `/workspace` as a Static Release, so assets are served from the edge with no Runtime Instances.

> **Note: Older image ids**
>
> Older image ids still work as `template_id` and boot the main image.
> The [Sandbox Templates API](/docs/api-reference/sandbox-templates) lists them.

---

## New desktop

A durable Linux desktop (Xfce and Firefox) with screenshot, click, type, and streaming, plus human takeover.
It is a **Computer** with `template_type: "miosa-desktop"`, not a sandbox template.

**Python**

```python
computer = client.computers.create(
    name="my-desktop",
    template_type="miosa-desktop",
    size="small",
)
```

**TypeScript**

```typescript
const computer = await miosa.computers.create({
  name: 'my-desktop',
  templateType: 'miosa-desktop',
  size: 'small',
});
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/computers \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{ "name": "my-desktop", "template_type": "miosa-desktop", "size": "small" }'
```

See [Computers](/docs/computers/overview) for the desktop surface, and [Embed a live desktop](/docs/computers/embedding) to put it in your product.

---

## Custom templates

When the built-in catalog does not fit  -  proprietary toolchains, specific CUDA/GPU stacks, pre-loaded datasets  -  you can build and register your own template.

1. Write a `Dockerfile` for your image.
2. Build it using the custom sandbox template API (or the CLI: `miosa template build`).
3. Verify the image boots correctly.
4. Register the slug so it is selectable in `template_id`.

See the [Custom Templates API Reference](/docs/api-reference/sandboxes) for the full endpoint spec, including image upload, smoke-test hooks, and versioning.

---

## BuildSpec primer

A **BuildSpec** is an optional JSON document you attach to a sandbox or Deployment to override the default install and start commands. It is the escape hatch for any template when defaults are not quite right:
```json
{
  "install": "pnpm install --frozen-lockfile",
  "build": "pnpm build",
  "start": "node .next/standalone/server.js",
  "env": {
    "NODE_ENV": "production",
    "PORT": "3000"
  }
}
```

Pass a BuildSpec at sandbox creation time or at publish time. Build-time commands (`install`, `build`) run inside the Builder; the `start` command determines how Runtime Instances are launched. Environment variables set here are merged with any variables already on the Deployment.

---

- [Sandboxes](/docs/develop/sandboxes): Full sandbox lifecycle - create, exec, files, snapshots, previews.

- [Publishing](/docs/docker-host/publishing): How the build pipeline turns a sandbox into a live deployment.

- [API Reference: Sandboxes](/docs/api-reference/sandboxes): Wire format for template_id, install/start commands, and BuildSpec.
