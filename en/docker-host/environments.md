---
slug: "docker-host/environments"
title: "Environments"
description: "Production, staging, preview/<source>: variable and secret scoping for deployments."
section: "Services & projects"
order: 1030
---

A **Deployment Environment** is the scope at which env vars, secrets, and data service bindings apply. Most projects have at least two: `production` and `staging`. Custom environments like `preview/feature-abc` are common for review-app workflows.

> **Note: Phase status**
>
> First-class `deployment_environments` records are part of the ongoing deployment refactor. Today, env vars are scoped per-deployment via `deployment_env_vars`. The shape and behavior here are the steady-state target.

## Defaults

Every project gets two environments at creation:

- `production`: what real users see
- `staging`: internal testing target before promoting to production

You can create additional environments (`preview/&lt;source&gt;`, `qa`, `canary`, etc.) as needed.

## What environments scope

| Item | Scope |
|---|---|
| Env vars | per environment |
| Secrets | per environment |
| Data service bindings (DATABASE_URL, REDIS_URL, etc.) | per environment |
| Domains | per environment (production gets the apex; staging gets a `staging.*` subdomain by convention) |
| Version promotion | per environment, promote a version to staging first, then to production |

The same deployment can run different versions in different environments. Common pattern:
```text

production:   version 17 active
staging:      version 18 active   ← being tested
preview/pr-42: version 19 active  ← single-feature preview
```

## Env vars
```http
POST /api/v1/deployments/:id/env
GET  /api/v1/deployments/:id/env
```

Env vars are stored encrypted at rest (ChaCha20-Poly1305) and injected into the sandbox during development and into runtime instances at boot. The list endpoint returns previews of values, not the raw values, by default.

**TypeScript**

```typescript
await miosa.deployments.env.set(deploymentId, {
  environment: "production",
  vars: {
    STRIPE_KEY: "sk_live_...",
    NEXT_PUBLIC_API_URL: "https://api.example.net",
  },
})

const preview = await miosa.deployments.env.list(deploymentId, {
  environment: "production",
})
// preview returns: { STRIPE_KEY: "sk_live_••••", NEXT_PUBLIC_API_URL: "https://api…" }
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/deployments/$DEP/env \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -d '{
    "environment": "production",
    "vars": { "STRIPE_KEY": "sk_live_..." }
  }'
```

## Secrets vs env vars

There's no strict UI distinction today: env vars are encrypted at rest regardless. The convention is:

- **Env vars**: configuration that can be retrieved as preview (truncated value).
- **Secrets**: same encryption, but marked `is_secret: true` so previews always show `••••` even to your own admins.

Future-state, secrets will support rotation and access-event audit logging.

## Data service bindings

When you create a Postgres / Redis / bucket via [Data](/docs/data/overview), you bind it to a specific deployment environment. MIOSA injects the credential into env at boot:
```text

production env:    DATABASE_URL=postgres://prod...
staging env:       DATABASE_URL=postgres://staging...
```

You don't ship two database URLs for one app; you ship one connection string variable, and MIOSA picks the right value based on which environment the runtime is booted in.

## Where to go next

- [Publishing](/docs/docker-host/publishing): environments are picked at publish time
- [Domains](/docs/docker-host/domains): attach a staging subdomain
- [Data](/docs/data/overview): environment-scoped credentials
