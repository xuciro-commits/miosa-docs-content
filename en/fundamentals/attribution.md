---
slug: "fundamentals/attribution"
title: "Ownership and external attribution"
description: "Canonical ownership fields, opaque external attribution, and usage rollup for platforms that resell MIOSA to their own customers."
section: "API reference"
order: 12005
---

Every MIOSA resource accepts canonical ownership fields and optional attribution fields:

| Field | Meaning |
|---|---|
| `workspace_id` / `workspace_slug` | The MIOSA workspace that owns the resource |
| `project_id` / `project_slug` | The MIOSA project that owns the resource |
| `external_workspace_id` | Your customer's account / tenant ID |
| `external_user_id` | The end-user inside that account |
| `external_project_id` | The project or app ID in your own database |

Every resource also accepts a free-form `metadata` object (up to 16 key-value string pairs). Use it to carry application-layer context  -  feature flags, plan tier, region, whatever your billing system needs  -  without bloating your own database with a MIOSA resource join.

Integrators use these fields for four main purposes:

- **Usage rollup**  -  query `GET /api/v1/sandboxes?external_workspace_id=acme-co` to aggregate compute time and costs for a downstream customer.
- **Per-end-user quotas**  -  enforce your own rate limits by counting active sandboxes per `external_user_id` before creating a new one.
- **Audit log filtering**  -  the audit log accepts `external_user_id` and `external_project_id` as filters so you can pull a customer-specific activity trail without exposing other customers' data.
- **Chargeback and billing**  -  `external_workspace_id` is the recommended key for billing rollup when you resell MIOSA compute to downstream customers. See the [Audit Log](/docs/api-reference/audit-log) and [Quotas](/docs/api-reference/quotas) references.

MIOSA stores canonical `workspace_id` and `project_id` on each resource. The external fields are stored as opaque strings. Both appear on list/read responses, and list endpoints accept them as filters:

**Python**

```python
# Create a sandbox attributed to a specific customer
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    workspace_slug="acme-co",
    workspace_name="Acme Co",
    project_slug="invoice-bot",
    project_name="Invoice Bot",
    external_workspace_id="acme-co",
    external_user_id="alice@example.net",
    external_project_id="invoice-bot",
    metadata={"plan": "pro", "region": "us-east"},
)

# List only sandboxes for that customer
sandboxes = client.sandboxes.list(
    workspace_id=sandbox.workspace_id,
)
```

**TypeScript**

```typescript
// Create
const sandbox = await miosa.sandboxes.create({
  templateId: 'miosa-sandbox',
  workspaceSlug: 'acme-co',
  workspaceName: 'Acme Co',
  projectSlug: 'invoice-bot',
  projectName: 'Invoice Bot',
  externalWorkspaceId: 'acme-co',
  externalUserId: 'alice@example.net',
  externalProjectId: 'invoice-bot',
  metadata: { plan: 'pro', region: 'us-east' },
});

// Filter
const sandboxes = await miosa.sandboxes.list({
  workspaceId: sandbox.workspaceId,
});
```

**cURL**

```bash
# Create
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "template_id": "miosa-sandbox",
    "workspace_slug": "acme-co",
    "workspace_name": "Acme Co",
    "project_slug": "invoice-bot",
    "project_name": "Invoice Bot",
    "external_workspace_id": "acme-co",
    "external_user_id": "alice@example.net",
    "external_project_id": "invoice-bot",
    "metadata": { "plan": "pro", "region": "us-east" }
  }'

# Filter
curl "https://api.miosa.ai/api/v1/sandboxes?workspace_id=550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer msk_u_..."
```

Filtering is always scoped to your MIOSA organization. You cannot see another organization's resources regardless of what workspace, project, or external IDs you pass.

See [Ownership and Attribution](/docs/platform/attribution) for the full filter spec and billing split by workspace/project.

- [Ownership and Attribution](/docs/platform/attribution): How white-label platforms scope customers, projects, and resources.
