---
slug: "services/previews"
title: "Preview environments"
description: "Open a pull request and MIOSA deploys it to its own environment with its own addresses, then removes it when the pull request closes."
section: "Services & projects"
order: 1011
---

A preview environment is a copy of your project's default environment that runs a pull request.
Open a pull request and MIOSA creates the environment, deploys the pull request's branch, and comments the addresses on the pull request.
Close or merge it and the environment is deleted.

## What happens

| Pull request event | What MIOSA does |
|---|---|
| Opened or reopened | Forks the default environment as `pr-<number>`, deploys every service from the pull request's branch, and comments the addresses |
| New commit pushed | Redeploys the services of `pr-<number>` |
| Closed or merged | Deletes `pr-<number>` and nothing else |

Each server service in the preview is named `<service>-pr-<number>`.
Variables are copied from the environment it was forked from.
A preview never changes the environment it came from.

## Turn it on

Open the project's **Settings** in the console and choose **Previews**.
Set the repository as `owner/name`, switch previews on, and save.
The page shows the webhook URL and the secret to paste into GitHub.

In the GitHub repository, add a webhook:

| Field | Value |
|---|---|
| Payload URL | The URL shown in Previews, `https://api.miosa.ai/api/v1/hooks/git/<project id>` |
| Content type | `application/json` |
| Secret | The secret from Previews |
| Events | Pull requests only |

The repository you set must match GitHub's `full_name` exactly, or deliveries are ignored.
A delivery is accepted only when its signature matches the secret.
The secret is write-only: it is never shown again, so keep a copy or set a new one.

## Cost control

| Setting | Default | Range |
|---|---|---|
| `max_open` | 3 | 1 to 20 |
| `ttl_hours` | 72 | 1 to 720 |

A fourth preview is refused while three are open.
A preview for a pull request nobody closed is deleted after 72 hours.
Raise or lower both in Previews.

## From the API

```text
GET    /api/v1/projects/:id/previews
PUT    /api/v1/projects/:id/previews
DELETE /api/v1/projects/:id/previews/:preview_id
```

`PUT` replaces the row, so send the whole object: `{repository, enabled, max_open, ttl_hours, secret}`.
A blank or absent `secret` keeps the one already stored.
The row you get back reports `has_secret`, never the secret, and the `webhook_path` to use.
These routes need the `projects:read` and `projects:write` scopes.

## Where to go next

- [Environments](/docs/services/environments): fork an environment by hand.
- [Variables](/docs/services/variables): what a preview copies.
