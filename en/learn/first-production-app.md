---
slug: "learn/first-production-app"
title: "Build and deploy a Next.js app"
description: "Build a small Next.js app in a Sandbox, preview it, publish it as an immutable Deployment, prove it is live, connect a custom domain, and roll back safely - with the miosa CLI 2.x and the SDKs."
section: "Get started"
order: 7
---

This tutorial takes a small Next.js application from an empty Sandbox to a verified production URL.

You will create a Sandbox, write the app, review a temporary Preview, publish an immutable Version, verify the stable Deployment URL, connect `app.example.com`, and learn rollback before you need it.

Do it now

Deploy your first app

Turn a repo or a sandbox into a stable production URL with immutable versions and rollback.

- [New deployment](https://miosa.ai/deploy/new): (opens the MIOSA console in a new tab)

miosa deploy create --from-sandbox my-box --wait

Publish a sandbox, a git repo (--git) or a local directory (--dir). Add --type docker_deploy for App Engine.

Next [Publishing guide](/docs/docker-host/publishing)

> **Note: What this tutorial preserves**
>
> Publishing does not destroy or convert the Sandbox.
> The Sandbox remains a development environment, while the Release and Deployment become independent production resources.

## What you will create

| Resource | Example | Purpose |
|---|---|---|
| Sandbox | `hello-nextjs` | Mutable place to create and test the application. |
| Preview | API-returned temporary URL | Review the live development server before publishing. |
| Deployment | `hello-nextjs` | Stable production identity and version history. |
| Platform URL | API-returned `public_url` | Immediate production access without customer DNS. |
| Custom domain | `app.example.com` | Optional customer-owned hostname attached to the Deployment. |

## Before you start

You need:

- A MIOSA User with access to the intended Organization and Workspace.
- A workspace API key stored only in a server or terminal environment.
- The MIOSA CLI installed (v2.2 or later).
- Permission to create Sandboxes, publish Deployments, and configure domains.
- Access to the DNS provider only if you complete the custom-domain section.

```bash
export MIOSA_API_KEY="msk_u_..."
miosa whoami
```

Confirm that the response identifies the expected Organization and Workspace before creating anything.

> **Caution: Do not continue in the wrong Organization**
>
> Resource IDs are not a substitute for checking the active customer boundary.
> If the CLI reports the wrong Organization or Workspace, correct that context before creating the Sandbox or Deployment.

## Step 1: Create the Sandbox

**CLI**

```bash
miosa create hello-nextjs \
  --template nextjs \
  --ttl 1h \
  --wait \
  --json
```

Save the returned Sandbox id:

```bash
export SANDBOX_ID="<sandbox-id>"
```

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const sandbox = await miosa.sandboxes.create({
  name: "hello-nextjs",
  templateId: "nextjs",
});
console.log(sandbox.id);
```

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sandbox = miosa.sandboxes.create(name="hello-nextjs", template_id="nextjs")
print(sandbox.id)
```

**Expected result:** the Sandbox reaches a usable running state.

If creation remains pending, inspect the returned capacity or readiness reason before retrying.
Do not create repeated Sandboxes while the first request is still progressing.

## Step 2: Create the Next.js application

Scaffold the app inside the Sandbox, then replace `app/page.tsx`.

**CLI**

```bash
miosa exec hello-nextjs --cwd /workspace -- bash -lc 'pnpm create next-app@latest hello-nextjs --typescript --eslint --app --no-tailwind --use-pnpm --yes'
```

Write the page file locally, then copy it into the Sandbox:

```bash
miosa files cp ./page.tsx hello-nextjs:/workspace/hello-nextjs/app/page.tsx
```

Build it:

```bash
miosa exec hello-nextjs --cwd /workspace/hello-nextjs --timeout 300 -- pnpm build
```

**TypeScript**

```typescript
const page = `export default function Home() {
  return (
    <main style={{ minHeight: "100vh", display: "grid", placeItems: "center" }}>
      <section style={{ textAlign: "center" }}>
        <p style={{ color: "#a1a1aa" }}>Deployed with MIOSA</p>
        <h1 style={{ fontSize: "clamp(3rem, 10vw, 7rem)", margin: 0 }}>Hello from MIOSA</h1>
      </section>
    </main>
  );
}`;

await sandbox.exec.run(
  "cd /workspace && pnpm create next-app@latest hello-nextjs --typescript --eslint --app --no-tailwind --use-pnpm --yes",
);
await sandbox.files.write("/workspace/hello-nextjs/app/page.tsx", page);

for await (const event of sandbox.exec.stream(
  "cd /workspace/hello-nextjs && pnpm build",
)) {
  console.log(event);
}
```

**Python**

```python
page = '''export default function Home() {
  return (
    <main style={{ minHeight: "100vh", display: "grid", placeItems: "center" }}>
      <h1>Hello from MIOSA</h1>
    </main>
  );
}'''

sandbox.exec.run(
    "cd /workspace && pnpm create next-app@latest hello-nextjs --typescript --eslint --app --no-tailwind --use-pnpm --yes",
)
sandbox.files.write_file("/workspace/hello-nextjs/app/page.tsx", page)

for event in sandbox.exec_stream("cd /workspace/hello-nextjs && pnpm build"):
    print(event)
```

**Expected result:** the build command exits with status `0` and Next.js reports a successful build.

## Step 3: Start and preview the application

Start the server as a background process so it keeps running after the CLI disconnects, then expose port 3000.

**CLI**

```bash
miosa process start hello-nextjs \
  --name web \
  --cwd /workspace/hello-nextjs \
  -- pnpm start --hostname 0.0.0.0 --port 3000

miosa preview create hello-nextjs 3000 --name web --json
```

**TypeScript**

```typescript
const preview = await sandbox.previews.create(3000);
console.log(preview.url);
```

**Python**

```python
preview = sandbox.previews.create(port=3000, label="app")
print(preview["url"])
```

Open the exact Preview URL returned by MIOSA.

**Expected result:** the page displays "Hello from MIOSA."

> **Tip: Preview is not production**
>
> The Preview reflects the current Sandbox process and files.
> It can change immediately and can disappear when the Sandbox stops.
> The stable production URL is created during publishing.

## Step 4: Publish a Deployment

`miosa deploy create` publishes a new Deployment and makes it live.
Give it the Sandbox as its source, the path the app lives at inside the Sandbox, and the command that serves it.

**CLI**

```bash
miosa deploy create \
  --from-sandbox hello-nextjs \
  --name hello-nextjs \
  --dir /workspace/hello-nextjs \
  --port 3000 \
  --run-command "pnpm start --hostname 0.0.0.0 --port 3000" \
  --wait \
  --json
```

Add `--type docker_deploy` to publish through Docker Host instead of the default runtime.

Save these values from the response:

```bash
export DEPLOYMENT_ID="<deployment-id>"
export PUBLIC_URL="<deployment-public-url>"
```

**TypeScript**

```typescript
const result = await sandbox.deploy({
  name: "hello-nextjs",
  path: "/workspace/hello-nextjs",
  port: 3000,
  runCommand: "pnpm start --hostname 0.0.0.0 --port 3000",
});

// Read public_url from the response. Do not construct a production hostname.
console.log(result);
```

**Python**

```python
result = sandbox.deploy(
    name="hello-nextjs",
    path="/workspace/hello-nextjs",
    port=3000,
    run_command="pnpm start --hostname 0.0.0.0 --port 3000",
)

# Read public_url from the response. Do not construct a production hostname.
print(result)
```

Always use the returned `public_url`.
Do not construct a production hostname from the slug, Organization name, or Workspace name.

**Expected result:** the response identifies the Deployment, candidate Release, promoted Version, and `public_url`.

## Step 5: Prove production

A successful build is not enough.
Confirm the Deployment resolved to a live Version and that the public URL answers.

```bash
miosa deploy get hello-nextjs --json
curl --fail --silent --show-error "$PUBLIC_URL" | grep "Hello from MIOSA"
miosa deploy observe hello-nextjs
```

**Expected result:** the Deployment reports a running state with a `public_url`, the public response contains "Hello from MIOSA", and `observe` shows the request.

## Step 6: Connect a custom domain

This step is optional.
The platform URL remains valid while DNS is being configured.

```bash
miosa deploy domain-add hello-nextjs app.example.com
```

MIOSA displays the required DNS record and verification target.
Copy those values exactly into the DNS provider.

For a normal subdomain, the record generally has this shape:

| Type | Host | Value | Proxying |
|---|---|---|---|
| `CNAME` | `app` | The target shown by MIOSA | Disabled until verification completes |

Do not copy `example.com` literally.
Use a hostname controlled by the customer and the target displayed in the live MIOSA domain flow.

Check public DNS, then verify:

```bash
dig app.example.com CNAME +short
miosa deploy domain-verify hello-nextjs <domain-id>
```

**Expected result:** the domain moves through DNS verification, certificate issuance, and routing until HTTPS is active.

> **Caution: Wildcard and exact domains are different**
>
> An Organization-level deployment domain can generate many application hostnames beneath one parent domain.
> An exact custom domain attaches one chosen hostname directly to one Deployment.
> Follow the record shown for the domain type you selected.
> See [Domains](/docs/docker-host/domains).

## Step 7: Publish an update safely

Change the heading in `app/page.tsx`, rebuild in the Sandbox, and publish again.

```bash
miosa exec hello-nextjs --cwd /workspace/hello-nextjs --timeout 300 -- pnpm build

miosa deploy create \
  --from-sandbox hello-nextjs \
  --name hello-nextjs \
  --dir /workspace/hello-nextjs \
  --port 3000 \
  --run-command "pnpm start --hostname 0.0.0.0 --port 3000" \
  --wait \
  --json
```

Reusing `--name` targets the existing Deployment, so its identity, stable URL, domains, bindings, and release history are preserved while a new Version is promoted.
Run the proof steps again after the new Version is promoted.

## Step 8: Understand rollback before you need it

List the Deployment versions:

```bash
miosa deploy versions hello-nextjs --json
```

If the new Version is unhealthy, return to the earlier ready one:

```bash
miosa deploy rollback hello-nextjs --to <version-id>
```

Rollback repoints the active Version in seconds without a rebuild.
It does not change the Deployment identity or require a new domain.
See [Rollback](/docs/docker-host/rollback).

## Common failures and recovery

| Symptom | Likely cause | Safe action | Proof after repair |
|---|---|---|---|
| Sandbox creation remains pending | Requested capacity is not ready or the request is still progressing. | Inspect the readiness reason and wait or select compatible capacity. | The original Sandbox reaches a usable state. |
| Preview returns an error | The Next.js process is stopped, bound to the wrong interface, or using another port. | Start it on `0.0.0.0:3000` with `miosa process start` and recreate the Preview. | The Preview returns "Hello from MIOSA." |
| Publish reports Docker Host is unavailable | The Workspace Docker Host Host is not active. | Confirm the Workspace has an active Docker Host Host, then retry the same `miosa deploy create ... --type docker_deploy`. | Publish returns a Deployment and `public_url`. |
| Build succeeds but the application is not live | Promotion, container health, or routing proof has not completed. | Inspect the publish result and run `miosa deploy get` and `miosa deploy observe`. | The public URL responds with the expected content. |
| DNS remains pending | The record has not propagated, uses the wrong host, or points to the wrong target. | Compare public DNS with the exact record shown by MIOSA and keep proxying disabled during verification. | `miosa deploy domain-verify` reports DNS verified and TLS active. |
| The custom hostname opens another application | The hostname was attached in the wrong Organization or to the wrong Deployment. | Stop and inspect ownership and route bindings before changing DNS again. | The domain resolves to the intended Deployment id and active Version. |
| A new publish causes a regression | The candidate passed build but application behavior is incorrect. | Roll back to the earlier ready Version. | The stable URL serves the previous known-good content. |

## Completion checklist

- [ ] The Sandbox exists in the intended Organization and Workspace.
- [ ] The Next.js production build exits successfully.
- [ ] The Preview displays "Hello from MIOSA."
- [ ] Publishing returns a Deployment id and API-selected `public_url`.
- [ ] The public URL returns the expected content.
- [ ] The custom domain is verified and TLS-active, if configured.
- [ ] An earlier ready Version is identifiable for rollback.
- [ ] The Sandbox can be removed without deleting the Deployment.

## Continue from here

- [Add Postgres](/docs/data/postgres): Bind durable application data without putting a database inside the application container.
- [Add secrets](/docs/security/secrets): Provide sensitive runtime configuration without committing it to source.
- [Understand publishing](/docs/docker-host/publishing): Learn the full Release, Version, promotion, proof, and retry contract.
- [Operate domains](/docs/docker-host/domains): Configure deployment domains, exact custom domains, verification, TLS, and recovery.

## Next steps

- [Publish from a sandbox](/docs/docker-host/publishing): Static, dynamic, and App Engine publishes with proof.
- [Attach a custom domain](/docs/docker-host/domains): DNS verification, TLS, and route caching.
- [Roll back in seconds](/docs/docker-host/rollback): Re-point a deployment at an older version with no rebuild.

- [Return to Learning Paths](/docs/learn): Choose the agent, desktop, or white-label journey next.
