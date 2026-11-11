---
slug: "api-reference/sandbox-templates"
title: "Sandbox Templates API"
description: "The templates that stage content inside the one sandbox image, the legacy image ids that are aliases, and how to create a sandbox from a template."
section: "API reference"
order: 12026
---

Sandbox templates are pre-built images with toolchains, dependencies, and configuration baked in. Launching from a template is faster than installing packages at runtime and ensures reproducible environments.

Every sandbox boots the same main image. A template is content staged inside that image at create time, never a separate image. See the [Templates guide](/docs/develop/templates).

> **Note**
>
> Templates are read-only marketplace items managed by MIOSA. To build your own custom templates, see the [Templates guide](/docs/develop/templates).

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/sandbox-templates` | List the templates available to you |
| `GET` | `/api/v1/sandbox-templates/{id}` | Get a specific template |
| `POST` | `/api/v1/sandboxes` | Create a sandbox from a template (via `template_id`) |

---

## List templates

**`GET /api/v1/sandbox-templates`**

Returns the built-in catalog plus any custom templates your workspace has built, as a JSON array.
Each row carries the template `slug`, its `source` (`builtin` or `workspace`), and the registry `status`.

Alias ids are left out by default. Pass `include_aliases=true` to include them; each alias is nested on the row it resolves to, under `aliases`.

```bash
curl "https://api.miosa.ai/api/v1/sandbox-templates?include_aliases=true" \
  -H "Authorization: Bearer msk_u_..."
```

---

## Listed templates

Five templates are listed, and all five run the one image:

| `template_id` | Kind | What it is |
|---|---|---|
| `miosa-sandbox` | `image` | The default sandbox. Everything inside, and what you get when you omit `template_id`. |
| `nextjs` | `starter` | Stages a Next.js app in the sandbox and starts it on port 3000. |
| `nextjs-postgres` | `starter` | A Next.js app with Postgres client tooling on port 3000; bind `DATABASE_URL`. |
| `static-html` | `starter` | Static HTML, CSS, and JavaScript served from `/workspace` on port 8000. |
| `fastapi-auth` | `starter` | A FastAPI starter with signup, login, me, and dashboard routes, on port 8000. Its health check does not work yet. |

The four starter templates add only content inside the image: the scaffold, the install command, and the start command.
Templates your workspace builds are returned with kind `custom`.
`miosa-desktop` is a separate Computer product, not a sandbox template; see [Computers](/docs/computers/overview).

---

## Aliases

Alias ids are never listed, and still work as `template_id`.

These resolve to `miosa-sandbox`, the plain image:

`miosa-sandbox-prod-1`, `miosa-sandbox-docker`, `agent-browser`, `instant-code`, `bookworm`, `debian-12`, `docker`, `sandbox-docker`, `browser-agent`, `code`, `instant`, `instant-node`, `node-20`, `postgres-15`, `postgres-16`, `ubuntu-22.04`, `ubuntu-24.04`.

These resolve to a starter, which stays listed:

`static` to `static-html`; `next-postgres` and `fullstack-next-postgres` to `nextjs-postgres`; `python-auth` and `fastapi-auth-starter` to `fastapi-auth`.

---

## Create a sandbox from a template

**`POST /api/v1/sandboxes`**

```json
{
  "template_id": "nextjs",
  "size": "small"
}
```

Omit `template_id` for a sandbox with nothing staged, or pass a content template to stage a scaffold inside the same image.
For a desktop, create a Computer with `template_type: "miosa-desktop"`; see [Computers](/docs/computers/overview).

See the [Sandboxes API](/docs/api-reference/sandboxes) for the full creation reference.

---

## Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 | `not_found` | Template ID does not exist |

---

## Examples

**cURL**

```bash
# List templates, including the legacy alias ids
curl "https://api.miosa.ai/api/v1/sandbox-templates?include_aliases=true" \
  -H "Authorization: Bearer msk_u_..."

# Get one template
curl https://api.miosa.ai/api/v1/sandbox-templates/nextjs \
  -H "Authorization: Bearer msk_u_..."

# Create a sandbox with the Next.js template staged
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"template_id": "nextjs", "size": "small"}'
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Browse templates
templates = client.sandbox_templates.list()
for t in templates:
    print(t.slug, "-", t.description)

# Create the default sandbox
sandbox = client.sandboxes.create(template_id="nextjs", size="small")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Browse
const templates = await client.sandboxTemplates.list();
for (const t of templates) {
  console.log(t.slug, '-', t.description);
}

// Create a sandbox with the Next.js template staged
const sandbox = await client.sandboxes.create({
  templateId: 'nextjs',
  size: 'small',
});
```
