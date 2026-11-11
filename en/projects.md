---
slug: "projects"
title: "Projects"
description: "Group the resources for one app, site, or deliverable inside a Workspace, without becoming a runtime."
section: "Services & projects"
order: 1000
---

A **Project** is the durable product boundary inside one Workspace.
It groups the resources for one application, site, workflow, or customer deliverable, and it never runs code itself.
Every resource that belongs to the product carries the project's `project_id`, so ownership, deployment history, domains, members, and usage all roll up to one boundary.

A Project always belongs to exactly one Workspace, and a Workspace belongs to one Organization.
The Workspace is required: creating a Project with no workspace fails with `WORKSPACE_REQUIRED`.

## 1. Create a project

```bash
miosa project create "Lead Magnet" \
  --workspace-id "$MIOSA_WORKSPACE_ID" \
  --description "Customer lead capture app"

miosa project list
miosa project get lead-magnet
```

## 2. Add resources

```bash
# Create a sandbox in the workspace, then publish its output.
miosa create lead-magnet-dev --workspace acme-corp --wait
miosa deploy create --from-sandbox lead-magnet-dev --wait
```

## 3. Deploy

```bash
miosa deploy create --git https://github.com/me/lead-magnet --branch main --wait
```

Publishing creates an immutable version under the Deployment and prints the URL.
Display the URL the API returns; never rebuild it from a slug.

## Pick what you need

| You want to | Page |
|---|---|
| See the project record, fields, and what lives inside it | [Project resources](/docs/projects/resources) |
| Run the full create, configure, deploy, and retire path | [Project lifecycle](/docs/projects/lifecycle) |
| Manage members, permissions, and billing attribution | [Members and usage](/docs/projects/members-and-usage) |

## Next

- [Project resources](/docs/projects/resources): The project record, what attaches to it, and how it maps to your own platform.
- [Project lifecycle](/docs/projects/lifecycle): Create, add resources, configure environments and secrets, deploy, and retire.
- [Members and usage](/docs/projects/members-and-usage): Members, permissions, usage attribution, and common mistakes.

## Related

- [Projects API](/docs/api-reference/projects): Create, resolve, update, and inspect Projects.
- [Publishing](/docs/docker-host/publishing): Move from mutable source to an immutable candidate.
- [Domains](/docs/docker-host/domains): Configure previews, generated app URLs, and exact hostnames.
- [Product model](/docs/concepts): How the Organization, Workspace, and Project layers relate.

Organizations, workspaces, and projects form one hierarchy: [Open the product model →](/docs/concepts).
