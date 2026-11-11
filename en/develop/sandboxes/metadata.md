---
slug: "develop/sandboxes/metadata"
title: "Metadata and labels"
description: "Attach arbitrary metadata to a sandbox at create time, read it back, and use external IDs for anything you need to query or bill on."
section: "Sandboxes"
order: 2010
---

Every sandbox record carries a `metadata` object you supply at create time. It is stored on the record and returned on reads. It is free-form: MIOSA stores it and never interprets it.

## Set metadata at create

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    metadata={"team": "growth", "ticket": "ENG-482", "run": "nightly"},
)
print(sandbox.metadata)   # {"team": "growth", "ticket": "ENG-482", "run": "nightly"}
```

**TypeScript**

```typescript
const sandbox = await client.sandboxes.create({
  templateId: 'miosa-sandbox',
  metadata: { team: 'growth', ticket: 'ENG-482', run: 'nightly' },
});
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"template_id": "miosa-sandbox", "metadata": {"team": "growth", "ticket": "ENG-482"}}'
```

The CLI sets the same field with repeated `--meta` flags:

```bash
miosa create my-box --meta team=growth --meta ticket=ENG-482
```

## Metadata or external IDs?

These are two different things, and the choice matters:

| Field | Queryable? | Use it for |
|---|---|---|
| `metadata` | Returned on read, not a list filter | Notes, tags, and pointers you read back with the record |
| `external_user_id`, `external_workspace_id`, `external_project_id` | Yes: list and usage filters | Anything you need to list, group, or bill on |

`GET /api/v1/sandboxes` filters by `workspace_id`, `project_id`, `external_workspace_id`, `external_user_id`, `external_project_id`, and `state`. It does not filter by `metadata`. If you need to enumerate every sandbox for one of your customers, use an external ID, not a metadata key.

> **Tip: Attribution flows downstream**
>
> The three `external_*` fields propagate to every usage record, audit event, and derived resource (previews, deployments) automatically, which is what per-tenant chargeback needs. Metadata does not. See [Ownership and attribution](/docs/platform/attribution).

## Per-tenant attribution

For per-tenant products, stamp the external IDs at create and read them back on every record:

```python
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    external_workspace_id="acme-corp",
    external_user_id="user-42",
    external_project_id="q3-dashboard",
    metadata={"requested_by": "jane@acme.example"},
)
```

Both fields are covered end to end in [Multi-tenant SaaS](/docs/cookbook/multi-tenant-saas).
