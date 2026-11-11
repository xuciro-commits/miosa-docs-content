---
slug: "migrate/vercel"
title: "Moving from Vercel"
description: "Import a Vercel project with miosa import vercel, review the plan, and what carries over."
section: "Optional"
order: 17029
---

`miosa import vercel` reads a Vercel project's `vercel.json` and writes a plan.
The plan names the MIOSA project, the `production` environment, one service, and the environment variable names the file declares.
Review the plan, then apply it.
Nothing is created until you apply.

## Import a project

```bash
npm i -g @miosa/cli
miosa import vercel --from-file vercel.json --name shop --repo https://github.com/acme/shop
miosa import vercel --from-file vercel.json --out plan.json
miosa import vercel --apply --plan plan.json
```

| Flag | Meaning |
|---|---|
| `--from-file` | The `vercel.json` to read |
| `--name` | The MIOSA project name |
| `--service` | The service name inside the project (default `web`) |
| `--repo` | The Git URL of the service's source |
| `--region` | `us-east`, `us-west` or `us-mia`; overrides the file |
| `--into` | Import into an existing MIOSA project |
| `--apply` | Create what the plan says |
| `--resume` | Continue an interrupted apply |

## What carries over

| Vercel | MIOSA |
|---|---|
| The app | A [service](/docs/services/overview) built from the repository |
| Region | The service region, when the file names one MIOSA has |
| Environment variable names | [Variables](/docs/services/variables) with no values |

Values are never copied.
Set each value on the service, then deploy.

## What does not carry over

The plan lists each of these with the reason: rewrites, redirects, headers, builds, the `functions` block, `cleanUrls`, `trailingSlash`, `buildCommand`, `installCommand`, `outputDirectory` and `crons`.
Recreate what you need as [service settings](/docs/services/services).

## Where to go next

- [Services quick start](/docs/services/quickstart)
- [Moving from Railway](/docs/migrate/railway)
