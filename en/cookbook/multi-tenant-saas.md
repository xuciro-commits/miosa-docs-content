---
slug: "cookbook/multi-tenant-saas"
title: "Multi-tenant SaaS"
description: "Embed MIOSA into your SaaS - tag sandboxes per tenant, query usage per workspace, and mint short-lived preview share tokens so customers interact directly with their sandboxes."
section: "Platform builders"
order: 9014
---

`~12 min` `Python` `TypeScript`

> **Goal**: Embed MIOSA into a multi-tenant product. Every sandbox is tagged with your own workspace and user IDs. Usage rolls up per tenant for billing. End-users interact with their sandboxes directly from the browser without ever seeing your API key.
>
> **What you'll use**: Sandboxes, External Attribution, preview share tokens, Usage API

## What you'll build

An embedded sandbox experience inside your SaaS:

1. Your backend creates sandboxes tagged to the right customer.
2. Each tenant's usage is queryable for chargeback.
3. Your customer's browser gets a short-lived token to embed a live preview  -  no `msk_*` key anywhere client-side.

## Prerequisites

- A MIOSA workspace API key (`msk_u_*`)  -  see [API Keys](/docs/platform/api-keys)
- Node 22+ or Python 3.11+

## Step 1  -  Install and configure

**Python**

```bash
pip install miosa
export MIOSA_API_KEY="msk_u_..."
```

**TypeScript**

```bash
npm install @miosa/sdk
export MIOSA_API_KEY="msk_u_..."
```

## Step 2  -  Tag every sandbox with your IDs

Pass `external_workspace_id`, `external_user_id`, and `external_project_id` on every `sandboxes.create` call. MIOSA stores these on the resource row and propagates them to every usage record, audit event, and derived resource (previews, deployments) automatically.

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def create_sandbox_for_user(workspace_id: str, user_id: str, project_id: str):
    """Called from your request handler when a tenant starts a new sandbox."""
    sbx = client.sandboxes.create(
        template_id="miosa-sandbox",
        timeout_sec=3600,
        external_workspace_id=workspace_id,
        external_user_id=user_id,
        external_project_id=project_id,
    )
    return sbx

# Example: workspace "acme-corp", user "jane-doe", project "q3-dashboard"
sbx = create_sandbox_for_user("acme-corp", "jane-doe", "q3-dashboard")
print(f"Created sandbox {sbx.id} for acme-corp / jane-doe")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

async function createSandboxForUser(
  workspaceId: string,
  userId: string,
  projectId: string,
) {
  const sbx = await client.sandboxes.create({
    templateId: 'miosa-sandbox',
    timeoutSec: 3600,
    externalWorkspaceId: workspaceId,
    externalUserId: userId,
    externalProjectId: projectId,
  });
  return sbx;
}

// Example
const sbx = await createSandboxForUser('acme-corp', 'jane-doe', 'q3-dashboard');
console.log(`Created sandbox ${sbx.id} for acme-corp / jane-doe`);
```

> **Tip**
>
> Use whatever ID format your platform already has  -  UUIDs, slugs, integer strings. MIOSA stores them as text and never validates the format beyond a 255-character length limit.

## Step 3  -  List sandboxes per tenant

Filter the sandbox list by your own IDs to show a customer only their resources, or to enumerate all sandboxes for an account before billing.

**Python**

```python
# All sandboxes for a workspace
acme_sandboxes = client.sandboxes.list(external_workspace_id="acme-corp")

for sbx in acme_sandboxes:
    print(f"  {sbx.id}  user={sbx.external_user_id}  status={sbx.status}")

# Narrow to a single user
jane_sandboxes = client.sandboxes.list(
    external_workspace_id="acme-corp",
    external_user_id="jane-doe",
)
```

**TypeScript**

```typescript
// All sandboxes for a workspace
const acmeSandboxes = await client.sandboxes.list({
  externalWorkspaceId: 'acme-corp',
});

for (const sbx of acmeSandboxes) {
  console.log(`  ${sbx.id}  user=${sbx.externalUserId}  status=${sbx.status}`);
}

// Narrow to a single user
const janeSandboxes = await client.sandboxes.list({
  externalWorkspaceId: 'acme-corp',
  externalUserId: 'jane-doe',
});
```

## Step 4  -  Query usage per tenant

Pull a usage report grouped by `external_workspace_id` at the end of each billing period. Use this to generate invoices or display a cost dashboard inside your product.

**Python**

```python
# Monthly rollup  -  grouped by workspace
usage = client.usage.report(
    period={"start": "2026-05-01", "end": "2026-06-01"},
    group_by=["external_workspace_id"],
)

for row in usage:
    print(f"  {row['external_workspace_id']}: ${row['total_usd']:.4f}")

# Drill into a single workspace by user
acme_by_user = client.usage.report(
    period={"start": "2026-05-01", "end": "2026-06-01"},
    external_workspace_id="acme-corp",
    group_by=["external_user_id"],
)

for row in acme_by_user:
    print(f"  user={row['external_user_id']}: ${row['total_usd']:.4f}")
```

**TypeScript**

```typescript
// Monthly rollup  -  grouped by workspace
const usage = await client.usage.report({
  period: { start: '2026-05-01', end: '2026-06-01' },
  groupBy: ['external_workspace_id'],
});

for (const row of usage) {
  console.log(`  ${row.externalWorkspaceId}: $${row.totalUsd.toFixed(4)}`);
}

// Drill into a single workspace by user
const acmeByUser = await client.usage.report({
  period: { start: '2026-05-01', end: '2026-06-01' },
  externalWorkspaceId: 'acme-corp',
  groupBy: ['external_user_id'],
});

for (const row of acmeByUser) {
  console.log(`  user=${row.externalUserId}: $${row.totalUsd.toFixed(4)}`);
}
```

## Step 5  -  Mint a preview share token for direct access

Your customer's browser needs to embed a live preview. You cannot put `msk_*` in the frontend. Instead, your backend mints a short-lived **preview share token** and returns it to the client.

> **Note: Preview share tokens, not browser tokens**
>
> This step mints a **preview share token** with `previews.share`, scoped to one preview URL. A **browser access token** is a different credential: it is used to embed a live desktop stream. See [Browser access tokens](/docs/platform/browser-tokens) if that is what you need instead.

**Backend  -  mint the token:**

**Python (FastAPI)**

```python
from fastapi import FastAPI, Depends, HTTPException
from miosa import Miosa
import os

app = FastAPI()
client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def get_current_user(request) -> dict:
    # Your existing auth logic  -  returns { workspace_id, user_id }
    ...

@app.get("/api/sandboxes/{sandbox_id}/preview-token")
async def get_preview_token(sandbox_id: str, user=Depends(get_current_user)):
    # Verify the sandbox belongs to this user's workspace
    sbx = client.sandboxes.get(sandbox_id)
    if sbx.external_workspace_id != user["workspace_id"]:
        raise HTTPException(403, "Forbidden")

    # Ensure a preview is running
    previews = sbx.previews.list()
    if not previews:
        preview = sbx.previews.create(port=3000)
    else:
        preview = previews[0]

    # Mint a 1-hour preview share token
    share = sbx.previews.share(
        preview.id,
        ttl_seconds=3600,
    )

    return {"share_url": share["share_url"], "expires_at": share["expires_at"]}
```

**TypeScript (Express)**

```typescript
import express from 'express';
import { Miosa } from '@miosa/sdk';

const app = express();
const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

// Assumes req.user is populated by your auth middleware
app.get('/api/sandboxes/:sandboxId/preview-token', async (req, res) => {
  const { sandboxId } = req.params;
  const { workspaceId } = (req as any).user;

  // Ownership check
  const sbx = await client.sandboxes.get(sandboxId);
  if (sbx.externalWorkspaceId !== workspaceId) {
    res.status(403).json({ error: 'Forbidden' });
    return;
  }

  // Ensure a preview exists
  const previews = await sbx.previews.list();
  const preview =
    previews.length > 0
      ? previews[0]
      : await sbx.previews.create(3000);

  // Mint a 1-hour preview share token
  const share = await sbx.previews.share(preview.id, { ttl_seconds: 3600 });

  res.json({ shareUrl: share.share_url, expiresAt: share.expires_at });
});
```

**Frontend  -  embed the preview:**

```html
<!-- Fetch the token from your backend, then embed the iframe -->
<script>
  async function embedPreview(sandboxId) {
    const resp = await fetch(`/api/sandboxes/${sandboxId}/preview-token`);
    const { shareUrl, expiresAt } = await resp.json();

    const iframe = document.getElementById('preview-frame');
    iframe.src = shareUrl;

    // Re-mint before expiry so the session stays alive
    const expiryMs = new Date(expiresAt).getTime() - Date.now() - 60_000;
    setTimeout(() => embedPreview(sandboxId), expiryMs);
  }

  embedPreview('sbx_...');
</script>

<iframe
  id="preview-frame"
  sandbox="allow-same-origin allow-scripts allow-forms"
  style="width: 100%; height: 600px; border: 0;"
></iframe>
```

> **Caution**
>
> Never put `msk_*` in a `<script>` tag, environment variable exposed to the browser, or localStorage. The share token is the only credential that should reach the browser, and it grants access to exactly one preview for one hour.

## Step 6  -  Full integration wiring

Here is the complete server-side flow for a new-user onboarding request:

**Python**

```python
from miosa import Miosa
import os

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

def onboard_new_project(workspace_id: str, user_id: str, project_id: str) -> dict:
    """Called when a customer creates a new project in your SaaS."""

    # 1. Boot a tagged sandbox
    sbx = client.sandboxes.create(
        template_id="miosa-sandbox",
        timeout_sec=3600,
        external_workspace_id=workspace_id,
        external_user_id=user_id,
        external_project_id=project_id,
    )

    # 2. Write an initial project scaffold
    sbx.files.write(
        "/workspace/README.md",
        f"# Project: {project_id}\nWorkspace: {workspace_id}\n",
    )

    # 3. Start the dev server
    sbx.commands.run("python3 -m http.server 3000", cwd="/workspace", background=True)

    # 4. Create a preview
    preview = sbx.previews.create(port=3000)

    # 5. Mint a preview share token
    share = sbx.previews.share(preview.id, ttl_seconds=3600)

    return {
        "sandbox_id": sbx.id,
        "preview_share_url": share["share_url"],
        "expires_at": share["expires_at"],
    }
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

async function onboardNewProject(
  workspaceId: string,
  userId: string,
  projectId: string,
): Promise<{ sandboxId: string; previewShareUrl: string; expiresAt: string }> {
  // 1. Boot a tagged sandbox
  const sbx = await client.sandboxes.create({
    templateId: 'miosa-sandbox',
    timeoutSec: 3600,
    externalWorkspaceId: workspaceId,
    externalUserId: userId,
    externalProjectId: projectId,
  });

  // 2. Write an initial scaffold
  await sbx.files.write(
    '/workspace/README.md',
    `# Project: ${projectId}\nWorkspace: ${workspaceId}\n`,
  );

  // 3. Start dev server (detached)
  sbx.exec.stream('python3 -m http.server 3000', { cwd: '/workspace' });

  // 4. Create a preview
  const preview = await sbx.previews.create(3000);

  // 5. Mint a preview share token
  const share = await sbx.previews.share(preview.id, { ttl_seconds: 3600 });

  return {
    sandboxId: sbx.id,
    previewShareUrl: share.share_url,
    expiresAt: share.data.shareTokenExpiresAt,
  };
}
```

## Step 7  -  Set per-user quotas

Cap each user so they cannot exhaust your MIOSA allocation. Quotas take effect immediately.

**Python**

```python
# Free-tier: max 3 sandboxes, 1 concurrent, $5 credit cap
client.quotas.set(
    "jane-doe",
    max_sandboxes=3,
    max_concurrent=1,
    max_credit_cents=500,
)

# Pro-tier: higher limits
client.quotas.set(
    "jane-doe",
    max_sandboxes=20,
    max_concurrent=5,
    max_credit_cents=5000,
)
```

**TypeScript**

```typescript
// Quotas API via REST (SDK method landing in a future release):
await fetch(`https://api.miosa.ai/api/v1/quotas/external/jane-doe`, {
  method: 'PUT',
  headers: {
    Authorization: `Bearer ${process.env.MIOSA_API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ max_sandboxes: 3, max_concurrent: 1, max_credit_cents: 500 }),
});
```

## Step 8  -  Audit log

Pull a paginated audit log to meet compliance requirements or investigate support tickets.

**Python**

```python
# List recent sandbox create/destroy events
events = client.audit_log.list(
    resource_type="sandbox",
    limit=50,
)

for event in events:
    print(f"  {event.get('inserted_at')}  {event.get('action')}  {event.get('resource_id')}")
```

**TypeScript**

```typescript
const events = await client.auditLog.list({
  resource_type: 'sandbox',
  limit: 50,
});

for (const event of events) {
  console.log(`  ${event.inserted_at}  ${event.action}  ${event.resource_id}`);
}
```

## What you learned

- Three external ID fields (`external_workspace_id`, `external_user_id`, `external_project_id`) group every MIOSA resource by your own identifiers with no server-side MIOSA configuration.
- Usage reports group by the same IDs  -  direct input to your chargeback logic.
- Share tokens are resource-scoped, expirable, and the only credentials that should reach a browser. The `msk_*` key stays on the backend at all times.
- Attribution is inherited: a deployment published from a tagged sandbox carries the same external IDs automatically.
- `client.quotas.set` caps end-users before they exhaust your allocation  -  call it on plan changes, not on every request.
- `client.auditLog.list` gives you a paginated event history for compliance and support.

## Next

- [Per-user quotas and billing](/docs/cookbook/per-user-quotas-billing): Full walkthrough of caps, monthly invoice rollups, and Stripe integration.
- [Embed sandbox iframe](/docs/cookbook/embed-sandbox-iframe): React, Vue, Svelte, and vanilla JS iframe embed with token TTL refresh.
- [Sandbox events in real time](/docs/cookbook/sandbox-events-realtime): Webhooks and SSE stream for sandbox lifecycle notifications.
