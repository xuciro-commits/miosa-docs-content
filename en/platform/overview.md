---
slug: "platform/overview"
title: "Platform builder guide"
description: "Run your users' agents, previews, and workspaces on MIOSA: architectures, costs, and provisioning patterns."
section: "Platform builders"
order: 9001
---

MIOSA gives you machines, previews, and deployments.
A platform is what you build on top of them so that other people can use them without ever learning how they work.

This guide is for builders of products like Lovable, Devin, or agent platforms.
In these products, your product makes a sandbox for each of your own users.

Such a platform needs:

- A sandbox per user or project: [Patterns](/docs/platform/patterns).
- A private preview URL of what the user builds: [Branding and domain setup](/docs/platform/white-label).
- An agent or harness working inside the machine: [Agents in your product](/docs/platform/agents-in-your-product).
- Deployments and Docker Host: [Deploy overview](/docs/docker-host/overview).
- White-label, tenants, scoped tokens, and external users: [Branding and domain setup](/docs/platform/white-label) and [Ownership and attribution](/docs/platform/attribution).
- Billing and caps: [Templates and sizing](/docs/platform/templates-and-sizing).

## Choose a shape

Every platform on MIOSA uses one model.
Three shapes cover most products.

| You are | Example | Who uses it | What is white-labeled |
|---|---|---|---|
| A SaaS shipping an agent product | **Lumen** | Lumen's customers | Everything: domains, branding, billing |
| An agency running client work | **Northwind Agency** | Northwind's clients | Client previews and delivery domains |
| A company with a team | **Acme Corp** | Acme's own staff and agents | Nothing. It is internal |

If you only need the internal case, jump to [Acme Corp: an internal platform](/docs/platform/patterns#acme-corp-an-internal-platform).

## Northwind Agency: client work

The full architectures - Lumen, Northwind Agency client work, and Acme Corp - are walked in [Patterns](/docs/platform/patterns).

## 1. Create a scoped platform key

Platform keys (`msk_p_*`) are for server-side automation.
Only tenant owners and administrators can create them.
Give each key only the scopes the backend needs, and optionally restrict it to one workspace.

**TypeScript**

```typescript
const key = await miosa.apiKeys.create({
  name: "Lumen backend prod",
  key_type: "platform",
  workspace_id: workspaceId,
  scopes: ["sandboxes:write", "deployments:write", "domains:write"],
})

console.log(key.key) // shown once, store it in your secret manager
```

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

key = miosa.api_keys.create(
    name="Lumen backend prod",
    key_type="platform",
    workspace_id=workspace_id,
    scopes=["deployments:write", "sandboxes:write", "domains:write"],
)

print(key["key"])  # shown once
```

The plaintext is returned exactly once and MIOSA stores only a hash.
Rotation, scopes, and audit are covered in [API keys](/docs/platform/api-keys).

## 2. Create a workspace per customer

Do this once, when the customer signs up, and store the returned IDs next to your own row.

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const workspace = await miosa.workspaces.create({
  name: "Northwind Agency",
  slug: "northwind-agency",
  externalWorkspaceId: "acct_123",
})
```

**Python**

```python
import miosa, os

client = miosa.Miosa(api_key=os.environ["MIOSA_API_KEY"])

workspace = client.workspaces.create(
    name="Northwind Agency",
    slug="northwind-agency",
    external_workspace_id="acct_123",
)
```

You can also skip this step.
If a create call carries `workspace_slug` or `external_workspace_id` and the workspace does not exist, MIOSA resolves or creates it.
The response returns the canonical `workspace_id` and `project_id`, so store those and reuse them.
Details in [Ownership and attribution](/docs/platform/attribution).

## 3. Give each user a machine

One sandbox or computer per end user, never shared across customers.

**TypeScript**

```typescript
const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  workspaceId: workspace.id,
  projectSlug: "booking-portal",
  projectName: "Booking Portal",
  externalUserId: user.id,         // your end user
  externalWorkspaceId: "acct_123", // your customer
})
```

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    workspace_id=workspace.id,
    project_slug="booking-portal",
    project_name="Booking Portal",
    external_user_id=user.id,
    external_workspace_id="acct_123",
)

# Everything in one customer's workspace
mine = client.sandboxes.list(workspace_id=sandbox.workspace_id)
```

Each machine has its own filesystem, network, and process tree.
Sizing, templates, and idle defaults are in [Templates and sizing](/docs/platform/templates-and-sizing).

## Pick what you need

| You want to | Page |
|---|---|
| Provision one sandbox per end user | [Patterns](/docs/platform/patterns) |
| Run an agent inside your users' machines | [Agents in your product](/docs/platform/agents-in-your-product) |
| Pick sizes, templates, and idle defaults | [Templates and sizing](/docs/platform/templates-and-sizing) |
| Put your brand on previews and apps | [Branding and domain setup](/docs/platform/white-label) |
| Attribute resources and usage to your customers | [Ownership and attribution](/docs/platform/attribution) |
| Meter usage and charge back | [Usage and billing](/docs/platform/usage-and-billing) |

## Next

- [Patterns](/docs/platform/patterns): The three architectures that cover most products, and how to provision each one.
- [Agents in your product](/docs/platform/agents-in-your-product): Give each user a machine with an agent or harness working inside it.
- [Connect agent accounts](/docs/guides/connect-agent-accounts): Connect a customer's Claude or ChatGPT plan, or their model key, from your own code.
- [Templates and sizing](/docs/platform/templates-and-sizing): Sizes, costs, templates, quotas, and idle defaults.
- [Internal Business OS](/docs/guides/internal-business-os): The internal-company version of this guide.

## Related

- [API keys](/docs/platform/api-keys): Key families, scopes, rotation, and audit.
- [Ownership and attribution](/docs/platform/attribution): Workspace and project ownership plus your external IDs.
- [Branding and domain setup](/docs/platform/white-label): Preview domains, deployment domains, custom domains, and branding.
- [Browser tokens](/docs/platform/browser-tokens): Short-lived tokens for previews and terminals in the end user's browser.
- [Connect agent accounts](/docs/guides/connect-agent-accounts): Keys and subscription plans, for your workspace or per end user.
- [Usage and billing](/docs/platform/usage-and-billing): Per-customer usage and chargeback on a single MIOSA bill.
- [Build gallery](/docs/build): Step-by-step guides for 20+ product shapes.

## Reference integration

Runnable examples live in [`examples/`](https://github.com/Miosa-osa/miosa/tree/main/examples).
See [`examples/agent-builds-static-app/`](https://github.com/Miosa-osa/miosa/tree/main/examples/agent-builds-static-app) for the "agent builds, previews, and deploys" pattern.
